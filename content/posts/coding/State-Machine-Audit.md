---
title: "为什么业务状态不能只靠几个布尔值"
date: 2026-08-21T10:20:00+08:00
---

# 为什么业务状态不能只靠几个布尔值：状态机、并发和审计

一个功能刚开始时，状态通常很简单。

例如订单只有：

```text
是否支付
```

于是数据库增加一个字段：

```text
is_paid = true / false
```

后来业务又需要：

- 是否取消
- 是否发货
- 是否退款
- 是否关闭

表里逐渐出现很多布尔值。

每个字段单独看都很清楚。

组合起来以后，却可能产生根本说不通的状态。

---

# 多个布尔值会产生非法组合

假设订单拥有：

```text
is_paid
is_cancelled
is_shipped
```

三个布尔值一共可以组合出八种情况。

其中可能出现：

```text
is_paid = false
is_cancelled = true
is_shipped = true
```

这表示订单没有支付、已经取消，却又已经发货。

业务上通常不允许这种情况。

字段越多，组合数量增长得越快。

如果每个接口只修改自己关心的一个布尔值，系统很难保证整体状态合理。

---

# 使用明确的业务状态

相比多个互相冲突的布尔值，可以为订单定义明确状态：

```text
pending_payment
paid
shipped
completed
cancelled
refunding
refunded
```

同一时刻，订单只处于其中一个主要状态。

这样很多非法组合从数据结构上就消失了。

但只有一个状态字段还不够。

系统还要规定：

> **每个状态允许进入哪些下一状态。**

---

# 状态不是任意修改的字符串

如果程序提供一个通用接口：

```text
UpdateOrderStatus(orderID, newStatus)
```

任何地方都可以把订单从 `pending_payment` 直接改成 `completed`。

状态字段虽然存在，业务规则仍然没有被保护。

更合理的方式是提供业务行为：

```text
PayOrder
CancelOrder
ShipOrder
ConfirmOrder
RefundOrder
```

每个行为检查当前状态。

例如：

```text
pending_payment → paid
pending_payment → cancelled
paid            → shipped
shipped         → completed
paid            → refunding
refunding       → refunded
```

不在允许列表里的变化应该被拒绝。

这就是状态机最基本的作用。

---

# 状态机表达的是业务规则

状态机不一定需要引入专门框架。

它可以是一张清晰的状态流转表，也可以是领域对象中的方法。

真正重要的是明确：

- 当前有哪些状态
- 哪些是初始状态
- 哪些是终止状态
- 每个动作允许在哪些状态执行
- 状态变化时还需要完成什么

例如取消订单不仅修改状态。

还可能需要：

- 记录取消原因
- 释放库存
- 关闭支付
- 发送事件

所以状态变化通常代表一次完整业务行为，而不是单纯更新一个字段。

---

# 并发会让正确的代码产生错误结果

假设订单当前是 `paid`。

两个请求同时到达：

```text
请求 A：发货
请求 B：退款
```

它们都先查询订单，看到状态是 `paid`。

请求 A 判断可以发货。

请求 B 判断可以退款。

随后两个请求分别更新状态。

最后写入的请求会覆盖前一个请求。

可能出现：

- 商品已经发出，订单却显示退款中
- 退款已经开始，订单却显示已发货

单个请求的判断都正确。

问题发生在两个请求使用了同一份旧状态。

---

# 条件更新保护状态流转

更新状态时，可以把旧状态也放进数据库条件：

```sql
UPDATE orders
SET status = 'shipped'
WHERE id = ? AND status = 'paid';
```

如果另一个请求已经把订单改成 `refunding`，这次更新就不会命中任何数据。

应用检查受影响行数后，可以知道：

> 当前状态已经变化，本次操作不能继续。

这种方式把“只有已支付订单可以发货”的规则带到了真正写入数据的位置。

它可以避免查询和更新之间的并发窗口。

---

# 版本号可以处理更通用的并发更新

有些对象不只是修改状态，还会修改其他字段。

可以增加一个版本号：

```text
version = 10
```

更新时执行：

```sql
UPDATE orders
SET status = ?, version = 11
WHERE id = ? AND version = 10;
```

只有仍然持有版本 10 的请求能够成功。

其他请求发现版本已经变化，就需要：

- 重新读取最新数据
- 放弃当前操作
- 根据业务决定是否重试

这种方式通常叫做乐观并发控制。

它假设冲突不会一直发生，但发生冲突时必须能够被检测出来。

---

# 消息顺序也会影响状态

假设订单先支付，随后退款。

系统依次产生两条消息：

```text
订单已支付
订单已退款
```

网络和重试可能让消费者先收到退款消息，后收到支付消息。

如果消费者只是无条件覆盖状态，订单最后可能重新变成已支付。

因此消费消息时不能只看消息内容。

还可以检查：

- 当前状态是否允许变化
- 消息版本是否比当前版本新
- 这个事件是否已经处理过

状态机负责判断变化是否合法。

版本和幂等标识负责识别旧消息与重复消息。

---

# 当前状态不能回答全部问题

假设订单现在是 `cancelled`。

只看当前状态，我们不知道：

- 谁取消了订单
- 为什么取消
- 取消前是什么状态
- 什么时候取消
- 是用户操作还是系统自动关闭

线上排查和业务审计往往需要这些信息。

因此除了当前状态，还可以保存状态变化历史。

例如：

```text
order_id      1001
from_status   pending_payment
to_status     cancelled
action        cancel_order
operator      user-88
reason        user_requested
request_id    cancel-1001
created_at    ...
```

---

# 审计记录和普通日志不同

普通日志主要用于技术排查。

它可能被：

- 采样
- 压缩
- 过期删除
- 调整格式

审计记录则是业务数据的一部分。

它需要稳定回答：

> 谁在什么时候，以什么原因，执行了什么业务操作？

因此重要审计数据通常需要：

- 结构化存储
- 明确保留周期
- 控制修改权限
- 避免记录敏感明文
- 能通过业务 ID 查询

不能只依赖某一行文本日志长期保存业务历史。

---

# 状态和历史应该一起提交

如果先更新订单状态，再写审计记录，第二步失败时会出现：

- 订单已经取消
- 没有取消历史

如果先写历史，再更新状态失败，则会出现一条没有真正发生的记录。

当状态和历史位于同一个数据库时，通常应该放在同一个本地事务里。

这样：

> **状态变化和变化记录要么一起成功，要么一起失败。**

事务提交以后，再通过消息处理其他可以最终一致的动作。

---

# 失败也可能是一种业务状态

有些长流程不是一次事务可以完成的。

例如退款可能需要等待第三方支付平台返回结果。

状态可以设计为：

```text
refund_pending
refunding
refunded
refund_failed
```

失败状态并不一定代表系统彻底结束。

它还可能包含：

- 失败原因
- 已重试次数
- 下一次重试时间
- 是否需要人工处理

把失败作为明确状态保存以后，系统才能知道哪些任务需要恢复。

如果失败只存在一条日志里，定时补偿任务就很难可靠找到它们。

---

# 不要创建无法维护的状态爆炸

状态机也不是状态越多越好。

如果把每一个细节都组合进一个字段，可能出现：

```text
paid_but_not_notified_waiting_inventory
paid_and_notified_inventory_failed
```

状态会快速膨胀。

不同生命周期可以适当拆开。

例如：

- 订单履约状态
- 支付状态
- 退款状态
- 物流状态

它们之间通过业务规则协作。

拆分的前提是明确哪些组合合法，以及谁负责跨状态规则。

不要用一个巨大状态表达所有事情，也不要用很多独立字段却不定义它们之间的关系。

---

# Context 连接一次完整的 Kratos 调用

Kratos 生成的 Service 方法会接收一个 `context.Context`。

这个 Context 应该沿着整条调用链继续向下传递：

```text
Middleware
   ↓
Service
   ↓
Biz Usecase
   ↓
Data Repo
   ↓
数据库和下游 RPC
```

Context 主要携带三类信息：

- Deadline：这次调用最晚什么时候结束
- Cancellation：调用方已经放弃时，通知下游停止工作
- Request-scoped Value：只属于这一次调用的上下文值

例如 Service 不应该重新创建一个 `context.Background()`：

```go
func (s *OrderService) ShipOrder(
	ctx context.Context,
	req *v1.ShipOrderRequest,
) (*emptypb.Empty, error) {
	if err := s.uc.Ship(ctx, req.Id, req.OperatorId); err != nil {
		return nil, err
	}
	return &emptypb.Empty{}, nil
}
```

继续传递原来的 `ctx`，数据库才能感知接口超时和客户端取消。

如果中途换成 `context.Background()`，上游取消以后，下面的 SQL 仍然可能继续执行。

---

# Context 不是一个通用参数 Map

Context Value 适合传递一次请求范围内、跨越多层调用的附加状态。

例如：

- Trace 信息
- 当前调用身份
- 事务句柄

普通业务参数仍然应该明确写进函数签名：

```go
Ship(ctx context.Context, orderID, operatorID int64) error
```

不应该全部藏进 Context：

```go
// 不推荐：调用方无法从签名知道需要哪些业务参数
Ship(ctx context.Context) error
```

使用 Context 时还应该遵守：

- 作为函数第一个参数传入
- 不传 `nil`
- 通常不要长期保存在 Struct 中
- 使用私有类型作为 Value Key，避免不同包发生冲突
- 派生了 Timeout 或 Cancel Context 后及时调用 `cancel`

Context 描述的是一次操作的生命周期。

它不应该变成隐藏依赖的容器。

---

# 事务边界应该由 Biz 决定

状态更新和审计记录为什么必须在同一个事务里？

因为业务规定：

> **订单状态发生变化时，必须同时留下状态历史。**

这是业务规则，所以应该由 Biz 用例决定事务范围。

Data 负责实现事务机制，但不应该擅自决定一个 Repo 方法和哪些其他操作共同提交。

Biz 可以定义自己需要的事务能力：

```go
// internal/biz/order.go
package biz

import (
	"context"

	"github.com/go-kratos/kratos/v3/errors"
)

type Transaction interface {
	InTx(context.Context, func(context.Context) error) error
}

type OrderStatus string

const (
	OrderPaid    OrderStatus = "paid"
	OrderShipped OrderStatus = "shipped"
)

type OrderTransition struct {
	OrderID int64
	From    OrderStatus
	To      OrderStatus
	ActorID int64
	Reason  string
}

type OrderRepo interface {
	Transit(context.Context, OrderTransition) error
}

type OrderHistoryRepo interface {
	CreateHistory(context.Context, OrderTransition) error
}

var ErrOrderStateConflict = errors.Conflict(
	"ORDER_STATE_CONFLICT",
	"order state has changed",
)
```

这个 `Transaction` 接口定义在 Biz。

因为 Biz 是事务能力的使用方。

它只要求“在同一个事务里执行这组业务操作”，并不知道底层使用 `sql.Tx`、GORM 还是其他实现。

---

# Biz 操作开启事务

`Ship` 用例把状态更新和审计记录放进同一个事务回调：

```go
// internal/biz/order.go
package biz

type OrderUsecase struct {
	tx      Transaction
	orders  OrderRepo
	history OrderHistoryRepo
}

func (uc *OrderUsecase) Ship(
	ctx context.Context,
	orderID int64,
	operatorID int64,
) error {
	transition := OrderTransition{
		OrderID: orderID,
		From:    OrderPaid,
		To:      OrderShipped,
		ActorID: operatorID,
		Reason:  "merchant_shipped",
	}

	return uc.tx.InTx(ctx, func(ctx context.Context) error {
		if err := uc.orders.Transit(ctx, transition); err != nil {
			return err
		}
		return uc.history.CreateHistory(ctx, transition)
	})
}
```

这里的事务边界就是一个完整 Biz 操作：

```text
发货 = 修改订单状态 + 写入审计历史
```

任何一步失败，整个业务操作都回滚。

Repo 不再各自开启事务。

否则两个 Repo 可能分别提交，仍然无法保证业务原子性。

---

# Data 实现事务并通过 Context 传递

`internal/data` 负责把 Biz 的 `InTx` 翻译成 `database/sql` 事务：

```go
// internal/data/transaction.go
package data

import (
	"context"
	"database/sql"
)

type txKey struct{}

type DBTX interface {
	ExecContext(context.Context, string, ...any) (sql.Result, error)
}

func txFromContext(ctx context.Context) (*sql.Tx, bool) {
	tx, ok := ctx.Value(txKey{}).(*sql.Tx)
	return tx, ok
}

func (d *Data) InTx(
	ctx context.Context,
	fn func(context.Context) error,
) error {
	// 嵌套调用复用当前事务，这里不提供 Savepoint 语义。
	if _, ok := txFromContext(ctx); ok {
		return fn(ctx)
	}

	tx, err := d.db.BeginTx(ctx, nil)
	if err != nil {
		return mapDataError(err)
	}
	defer tx.Rollback()

	txCtx := context.WithValue(ctx, txKey{}, tx)
	if err := fn(txCtx); err != nil {
		return err
	}
	if err := tx.Commit(); err != nil {
		return mapDataError(err)
	}
	return nil
}

func (d *Data) DB(ctx context.Context) DBTX {
	if tx, ok := txFromContext(ctx); ok {
		return tx
	}
	return d.db
}
```

`txKey` 是 Data 包的私有类型。

Biz 只负责把派生后的 Context 继续传给 Repo，无法直接操作 `*sql.Tx`。

这样既保持了依赖方向，也让同一业务操作中的多个 Repo 自动使用同一个事务。

---

# Repo 从 Context 选择 DB 或 Tx

订单 Repo 不再调用 `BeginTx`。

它只从 Data 取得当前 Context 对应的执行器：

```go
// internal/data/order.go
package data

func (r *orderRepo) Transit(
	ctx context.Context,
	t biz.OrderTransition,
) error {
	result, err := r.data.DB(ctx).ExecContext(ctx, `
		UPDATE orders
		SET status = ?, version = version + 1
		WHERE id = ? AND status = ?`,
		t.To, t.OrderID, t.From,
	)
	if err != nil {
		return mapDataError(err)
	}

	n, err := result.RowsAffected()
	if err != nil {
		return mapDataError(err)
	}
	if n != 1 {
		return biz.ErrOrderStateConflict
	}
	return nil
}
```

审计 Repo 使用同样的方式：

```go
func (r *orderHistoryRepo) CreateHistory(
	ctx context.Context,
	t biz.OrderTransition,
) error {
	_, err := r.data.DB(ctx).ExecContext(ctx, `
		INSERT INTO order_state_history
		(order_id, from_status, to_status, actor_id, reason)
		VALUES (?, ?, ?, ?, ?)`,
		t.OrderID, t.From, t.To, t.ActorID, t.Reason,
	)
	if err != nil {
		return mapDataError(err)
	}
	return nil
}
```

在 `InTx` 回调内，`DB(ctx)` 返回同一个 `*sql.Tx`。

在普通调用中，它返回连接池 `*sql.DB`。

Repo 不需要知道当前是否处于事务中，也不会意外绕过 Biz 定义的事务边界。

---

# Context 还能传递取消和超时

事务 Context 不只携带 `*sql.Tx`。

它仍然继承原始请求的 Deadline 和 Cancellation。

如果 Kratos 上游请求已经超时：

- `ctx.Done()` 会被关闭
- `ExecContext` 会把取消信号传给支持 Context 的数据库驱动
- `InTx` 返回错误
- 延迟执行的 `Rollback` 释放事务资源

具体 SQL 能否立即停止取决于数据库和驱动实现。

因此不要为了开启事务创建一个新的 `context.Background()`。

应该始终从当前业务 Context 派生事务 Context。

同时也不要把事务 Context 保存下来，在请求结束后继续使用。

它只属于当前这一次 Biz 操作。

这里体现了 Kratos 分层的责任：

- Service 传递请求 Context，不重新创建调用生命周期
- Biz 定义完整业务操作和事务边界
- Data 实现 `InTx`，并把具体事务绑定到派生 Context
- Repo 使用 Context 中的事务执行 SQL

> 代码采用 `database/sql` 展示核心思路。使用 GORM、Ent 等组件时实现方式不同，但事务边界仍然应该由 Biz 用例决定。

---

# 小结

多个布尔值看起来简单，却会产生大量非法组合。

明确状态加上允许的流转规则，才能表达真实业务生命周期。

状态设计需要同时考虑：

- 哪些动作允许在哪些状态执行
- 并发请求怎样避免互相覆盖
- 重复和乱序消息怎样处理
- 失败以后怎样恢复
- 状态历史怎样审计

条件更新和版本号负责发现并发冲突。

状态机负责判断业务变化是否合法。

审计记录负责回答过去发生了什么。

真正可靠的业务状态不是一个可以随意修改的字符串。

而是：

> **每一次变化都有前提、有结果、有记录，并且在并发下仍然不能越过业务规则。**
