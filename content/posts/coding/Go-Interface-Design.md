---
title: "为什么 Go 的接口通常由使用方定义"
date: 2026-08-21T10:40:00+08:00
---

# 为什么 Go 的接口通常由使用方定义：从 Interface 到 Kratos 依赖反转

在一些面向对象语言里，通常先定义一个接口：

```text
OrderRepository
```

再明确声明某个类实现它。

写 Go 时，很容易把同样的习惯带过来：

```text
IOrderService
OrderServiceImpl
IOrderRepository
OrderRepositoryImpl
```

结果每个 Struct 都有一个一模一样的 Interface。

代码看起来完成了面向接口编程。

实际上只是为每个类型增加了一层名字。

Go 的 Interface 更关注：

> **当前使用方真正需要哪些行为。**

而不是：

> **这个实现类型应该属于哪一套继承关系。**

---

# Go 的 Interface 是隐式实现的

在 Go 中，一个类型只要拥有接口要求的方法，就自动满足这个接口。

例如：

```go
package notice

type Sender interface {
	Send(message string) error
}

type EmailSender struct{}

func (EmailSender) Send(message string) error {
	// 发送邮件
	return nil
}
```

`EmailSender` 没有声明 `implements Sender`。

但它已经可以作为 `Sender` 使用：

```go
func Notify(sender Sender, message string) error {
	return sender.Send(message)
}

err := Notify(EmailSender{}, "order paid")
```

实现方甚至不需要知道 `Sender` 接口存在。

这种隐式满足让接口可以在真正需要它的地方再出现。

---

# 接口应该属于使用方

假设短信包提供一个具体类型：

```go
// package sms
type Client struct{}

func (c *Client) Send(phone, content string) error {
	return nil
}

func (c *Client) Balance() (int64, error) {
	return 0, nil
}
```

订单业务只需要发送短信。

那么订单包可以定义自己的最小接口：

```go
// package order
type MessageSender interface {
	Send(phone, content string) error
}

type Usecase struct {
	sender MessageSender
}
```

订单业务不需要依赖 `Balance`。

也不需要要求短信包专门提供一个和 `Client` 方法完全一致的大接口。

接口由使用方定义以后，表达的是：

> **订单业务只依赖发送消息这项能力。**

---

# 为什么不在实现方提前定义接口

一种常见写法是：

```go
// package sms
type Client interface {
	Send(phone, content string) error
	Balance() (int64, error)
	Close() error
}

type clientImpl struct{}
```

问题在于，不同使用方需要的能力可能不同。

订单只需要 `Send`。

财务系统只需要 `Balance`。

程序退出逻辑只关心 `Close`。

实现方提前定义的大接口，要求所有替代实现同时实现全部方法。

使用方定义小接口，则可以分别得到：

```go
type Sender interface {
	Send(phone, content string) error
}

type BalanceReader interface {
	Balance() (int64, error)
}

type Closer interface {
	Close() error
}
```

同一个 `*sms.Client` 可以同时满足它们。

但每个使用方只依赖自己需要的部分。

---

# 接口越大，抽象能力反而越弱

假设一个接口包含十几个方法：

```go
type UserRepository interface {
	Create(context.Context, *User) error
	Update(context.Context, *User) error
	Delete(context.Context, int64) error
	Find(context.Context, int64) (*User, error)
	List(context.Context) ([]*User, error)
	Count(context.Context) (int64, error)
	Export(context.Context) ([]byte, error)
	Import(context.Context, []byte) error
	Lock(context.Context, int64) error
	Unlock(context.Context, int64) error
}
```

想替换这个接口，就必须实现全部方法。

即使当前业务只需要查询一个用户，测试 Fake 也要补齐其他方法。

接口越小，能满足它的类型越多。

Go 标准库中的 `io.Reader` 只有一个方法：

```go
type Reader interface {
	Read(p []byte) (n int, err error)
}
```

文件、网络连接、压缩流和内存 Buffer 都可以满足它。

小接口并不意味着能力弱。

它表示调用方只要求最少的行为契约。

---

# 接口应该围绕调用场景拆分

不要为了追求“一方法接口”机械拆分所有代码。

如果几个方法总是在同一个场景中一起使用，把它们放在一个接口里也很自然。

例如一个事务需要：

```go
type Tx interface {
	Commit() error
	Rollback() error
}
```

判断接口大小时可以问：

- 使用方是否真的会一起调用这些方法
- 替代实现是否自然拥有这些能力
- 增加一个方法会影响多少实现
- 测试时是否被迫实现无关方法

接口应该围绕稳定行为组合。

不是围绕某个 Struct 当前碰巧拥有的全部方法组合。

---

# 接收接口，通常返回具体类型

Go 中经常使用一个经验：

> **函数接收接口，构造函数返回具体类型。**

例如：

```go
package sms

type Client struct {
	endpoint string
}

func NewClient(endpoint string) *Client {
	return &Client{endpoint: endpoint}
}
```

使用方再根据自己的需要定义接口：

```go
type Sender interface {
	Send(phone, content string) error
}
```

返回具体类型有几个好处：

- 实现方以后可以增加新方法
- 调用方可以自行选择需要哪些行为
- 不会过早承诺一个不成熟的抽象

但这不是绝对规则。

如果构造函数有意隐藏实现，而且调用方只应该看到稳定契约，返回接口也可以是合理设计。

关键不是背一句口诀。

而是明确接口由谁拥有，以及为什么需要隐藏具体类型。

---

# 不要为了 Mock 给每个 Struct 创建接口

下面这种接口通常没有提供真正抽象：

```go
type OrderService interface {
	Create(context.Context, *Order) error
	Get(context.Context, int64) (*Order, error)
}

type OrderServiceImpl struct {
	// ...
}
```

如果接口和实现始终一对一，并且没有使用方需要替代它，可以先直接使用具体类型。

等真实调用方出现以后，再从调用方式中提取最小接口。

为了测试，也应该优先在依赖的使用方定义接口。

不是让每个实现包提前猜测未来会怎样 Mock。

---

# 一个方法的接口有时可以直接使用函数

假设代码只需要发送一次通知：

```go
type Sender interface {
	Send(context.Context, string) error
}
```

也可以使用函数类型：

```go
type SendFunc func(context.Context, string) error

type Usecase struct {
	send SendFunc
}
```

测试时直接传入闭包：

```go
uc := Usecase{
	send: func(ctx context.Context, message string) error {
		got = message
		return nil
	},
}
```

如果这项能力有明确领域名称、存在多个实现，Interface 通常更清楚。

如果只是注入一个简单回调，函数可能更加直接。

---

# Interface 组合比继承更轻

Go 可以通过嵌入组合接口：

```go
type Reader interface {
	Read([]byte) (int, error)
}

type Writer interface {
	Write([]byte) (int, error)
}

type ReadWriter interface {
	Reader
	Writer
}
```

只需要读取的函数接收 `Reader`。

只需要写入的函数接收 `Writer`。

同时需要两者时才接收 `ReadWriter`。

这样调用方不会因为一个巨大父接口，被迫依赖自己根本不用的方法。

---

# Kratos 的 Repo 为什么定义在 Biz

Kratos Layout 中常见的依赖方向是：

```text
service → biz ← data
```

`biz` 是数据库能力的使用方。

所以 Repo 接口由 `biz` 定义：

```go
// internal/biz/order.go
package biz

import "context"

type Order struct {
	ID     int64
	Status string
}

type OrderRepo interface {
	FindByID(context.Context, int64) (*Order, error)
	Save(context.Context, *Order) error
}

type OrderUsecase struct {
	repo OrderRepo
}

func NewOrderUsecase(repo OrderRepo) *OrderUsecase {
	return &OrderUsecase{repo: repo}
}
```

这个接口描述订单业务真正需要的存储能力。

它没有暴露 SQL、ORM Model 或 Redis Client。

---

# Data 隐式实现 Biz 的接口

`internal/data` 不需要再定义一个 `OrderRepo` 接口。

它只实现 Biz 已经提出的行为：

```go
// internal/data/order.go
package data

import (
	"context"

	"example/internal/biz"
)

type orderRepo struct {
	data *Data
}

var _ biz.OrderRepo = (*orderRepo)(nil)

func NewOrderRepo(data *Data) biz.OrderRepo {
	return &orderRepo{data: data}
}

func (r *orderRepo) FindByID(
	ctx context.Context,
	id int64,
) (*biz.Order, error) {
	// 查询数据库并把 PO 转换成 DO
	return nil, nil
}

func (r *orderRepo) Save(
	ctx context.Context,
	order *biz.Order,
) error {
	// 把 DO 转换成 PO 后持久化
	return nil
}
```

这一行是编译期检查：

```go
var _ biz.OrderRepo = (*orderRepo)(nil)
```

它不会创建对象，也没有运行时开销。

如果 `*orderRepo` 少实现一个方法，编译会直接失败。

---

# 这和“返回具体类型”冲突吗

一般 Go 包更倾向让构造函数返回具体类型，由使用方定义接口。

Kratos Repo 是一个有意设计出的架构边界：

- 接口仍然由使用方 `biz` 定义
- 实现类型是 `data` 内部细节
- Wire 只需要把实现注入 Biz 契约

因此 `NewOrderRepo` 返回 `biz.OrderRepo` 是合理的。

它没有在实现方创建一个大而通用的接口。

而是在组装层明确隐藏 Data 的具体实现。

另一种写法是让构造函数返回 `*orderRepo`，再使用 Wire Bind 绑定到 `biz.OrderRepo`。

两种方式都能工作。

真正需要保持的是：

> **Biz 不反向依赖 Data 的具体类型。**

---

# 小接口让 Biz 测试更简单

测试订单用例时，不需要启动真实数据库。

可以实现 Biz 需要的最小 Repo：

```go
package biz_test

type fakeOrderRepo struct {
	order *biz.Order
	saved bool
}

func (f *fakeOrderRepo) FindByID(
	context.Context,
	int64,
) (*biz.Order, error) {
	return f.order, nil
}

func (f *fakeOrderRepo) Save(
	context.Context,
	*biz.Order,
) error {
	f.saved = true
	return nil
}
```

Fake 只需要实现当前用例依赖的方法。

如果 Repo 接口包含大量无关 CRUD，测试代码也会被迫跟着膨胀。

---

# 指针和值实现的接口可能不同

Go 的方法集需要特别注意。

如果方法使用指针接收者：

```go
type Store struct{}

func (*Store) Save() error {
	return nil
}
```

那么满足下面接口的是 `*Store`：

```go
type Saver interface {
	Save() error
}

var _ Saver = (*Store)(nil) // 正确
```

`Store` 值本身并不拥有指针接收者的方法集。

这也是编译期断言经常写成 `(*Type)(nil)` 的原因。

---

# Interface 的 Nil 陷阱

Interface 值内部可以简单理解成：

```text
动态类型 + 动态值
```

只有两者都为空时，Interface 才等于 `nil`。

下面的代码容易产生误解：

```go
type BizError struct{}

func (*BizError) Error() string { return "biz error" }

func execute() error {
	var err *BizError = nil
	return err
}

func main() {
	err := execute()
	fmt.Println(err == nil) // false
}
```

返回的 `error` 包含动态类型 `*BizError`，只是内部指针值为 `nil`。

因此它并不是一个 Nil Interface。

正确做法是没有错误时直接返回 `nil`：

```go
func execute() error {
	return nil
}
```

在 Kratos 中，如果把一个带类型的 Nil Error 返回到 Service，它仍然可能被错误中间件当成失败，污染日志和 Trace。

---

# Any 不是面向接口设计

`any` 是 `interface{}` 的别名。

它没有要求任何方法，所以所有类型都满足它。

下面的代码虽然使用了 Interface：

```go
func Save(value any) error
```

但它没有表达任何行为契约。

调用方和实现方只能在运行时通过类型断言猜测里面是什么。

真正的面向接口设计应该表达能力：

```go
type Saver interface {
	Save(context.Context) error
}
```

`any` 适合确实需要容纳任意值的底层工具。

它不能代替清晰的领域模型和方法签名。

---

# 泛型和 Interface 解决的问题不同

泛型适合表达：

> **一组类型可以执行同一种编译期算法。**

例如通用集合操作：

```go
func Contains[T comparable](items []T, target T) bool {
	for _, item := range items {
		if item == target {
			return true
		}
	}
	return false
}
```

普通 Interface 更适合表达：

> **运行时对象能够提供什么行为。**

例如：

```go
type OrderRepo interface {
	FindByID(context.Context, int64) (*Order, error)
}
```

泛型不能自动解决依赖反转。

Interface 也不适合拿来模拟所有容器算法。

选择时应该先判断自己要抽象的是类型算法还是对象行为。

---

# 什么时候不需要 Interface

下面这些情况可以先使用具体类型：

- 只有一个实现和一个调用方
- 调用方需要实现类型的大部分能力
- 接口只是完整复制 Struct 的所有方法
- 抽象边界还没有稳定
- 没有替换、隔离或测试需求

过早定义接口，会让每次增加方法都同时修改接口、实现和所有 Fake。

先从具体代码开始，等调用关系出现以后再提取接口，通常更符合 Go 的方式。

---

# 小结

Go 的 Interface 不是继承体系。

它表达的是使用方需要的一组行为。

设计时可以遵循几个判断：

- 接口尽量定义在使用方
- 接口只包含当前场景真正需要的方法
- 实现类型通过方法集隐式满足接口
- 不要为了 Mock 给每个 Struct 预先创建接口
- 普通构造函数优先返回具体类型
- 有意隐藏实现的架构边界可以返回接口
- 使用编译期断言检查关键实现关系
- 注意带类型的 Nil Interface
- 区分泛型的类型算法和 Interface 的运行时行为

Kratos 中 `biz` 定义 Repo、`data` 实现 Repo，正是这种思想的实际应用。

Biz 不关心使用 MySQL、内存还是远程服务。

它只声明完成业务用例真正需要的能力。

所以面向接口设计的重点不是项目里有多少个 Interface。

而是：

> **依赖是否指向稳定的业务需求，并且每个接口都只表达使用方真正需要的行为。**

> 代码示例采用 Go 当前语法和 Kratos v3 项目结构，省略了部分构造、错误映射和 Wire 组装代码。

延伸阅读：

- [Go Code Review Comments：Interfaces](https://go.dev/wiki/CodeReviewComments#interfaces)
- [Effective Go：Interfaces and other types](https://go.dev/doc/effective_go#interfaces_and_types)
