---
title: "为什么架构设计不是多分几层"
date: 2026-08-21T10:10:00+08:00
---

# 为什么架构设计不是多分几层：从依赖方向到中间件设计

提到架构设计时，经常会想到：

- 分层架构
- 微服务
- 事件驱动
- DDD
- 中间件

于是一个项目可能很快出现很多目录和模块。

但目录越多，不代表架构越好。

架构真正解决的是：

> **系统发生变化时，影响范围能不能被控制。**

如果修改一个业务规则需要同时修改接口、数据库、MQ 和多个公共组件，说明依赖边界并不清楚。

---

# 分层的目的是什么

一个常见服务可能分成：

```text
接口层
应用层
领域层
基础设施层
```

这些层不是为了让调用链更长。

而是分开不同变化原因。

例如：

- HTTP 参数格式变化，不应该影响业务规则
- 数据库从 MySQL 切换存储方式，不应该重写所有业务流程
- 业务规则变化，不应该要求修改 Trace 中间件

好的边界让不同类型的变化停留在自己的范围内。

---

# 依赖方向比目录位置更重要

即使代码放在不同目录，也可能互相随意引用。

例如领域层直接依赖：

- HTTP Request
- ORM Model
- Redis Client
- MQ SDK

这意味着业务规则已经和具体技术绑定。

技术组件一变化，领域代码就要一起变化。

更稳定的方向通常是：

```text
外部接口和基础设施
        ↓
应用用例和业务规则
```

业务层定义自己需要的能力。

基础设施层实现这些能力。

这样核心业务不需要知道请求来自 HTTP、定时任务还是消息队列。

---

# 不要为了抽象而抽象

抽象的价值是隔离真实变化。

如果一个组件只有一种实现，也没有变化可能，却提前创建很多接口、工厂和适配器，代码会更难理解。

判断是否需要抽象，可以先问：

- 这里是否有多个调用方
- 变化频率是否不同
- 是否需要隔离外部依赖
- 是否已经出现重复逻辑
- 测试是否需要替换实现

抽象应该来自已经存在的边界和变化。

不是来自对未来所有可能性的猜测。

---

# 单体不等于没有架构

很多系统一开始并不需要微服务。

一个模块清晰的单体应用，同样可以拥有良好架构。

例如在同一个进程里划分：

- 订单模块
- 库存模块
- 支付模块

模块之间通过清晰接口协作。

以后真的需要独立扩容或独立发布时，再拆成服务。

如果单体内部边界混乱，拆成微服务以后，只会把函数调用变成网络调用。

原来的耦合仍然存在，还增加了超时、重试和一致性问题。

---

# 中间件在解决什么问题

一次请求进入服务以后，很多逻辑和具体业务无关。

例如：

- 记录 Trace
- 身份认证
- 权限检查
- 限流
- 超时控制
- 异常恢复
- 指标统计

如果每个接口都重复编写这些逻辑，很容易出现不一致。

中间件允许把这些横切能力放在统一位置。

请求先经过一组中间件，再进入业务处理器。

响应则沿着原来的路径返回。

---

# 中间件像一层层包裹

假设中间件顺序是：

```text
Trace → Auth → Timeout → Handler
```

执行过程大致是：

```text
Trace 开始
  Auth 开始
    Timeout 开始
      Handler 执行
    Timeout 结束
  Auth 结束
Trace 结束
```

所以中间件不是一个简单列表。

它更像一层层包裹业务处理器。

越靠外的中间件：

- 越早看到请求
- 越晚看到响应
- 能覆盖越完整的执行过程

理解这一点以后，才能正确安排顺序。

---

# 中间件顺序会改变行为

假设 Trace 放在认证之后。

认证失败的请求可能没有完整 Trace。

如果指标统计放在限流之后，被限流的请求可能没有进入请求总量。

如果异常恢复放得太里面，外层中间件的 Panic 可能无法捕获。

因此中间件顺序需要根据希望观察和保护的范围决定。

一种关注可观测性的思路是：

```text
Trace 和请求标识
  ↓
指标与日志
  ↓
异常恢复
  ↓
限流
  ↓
认证与权限
  ↓
超时控制
  ↓
业务处理
```

把异常恢复放在 Trace 和日志里面，是为了让业务发生 Panic 时，Recovery 先把 Panic 转换成 Error，再让外层 Logging 和 Trace 看到这个错误。

如果把 Recovery 放在最外层，它能保护更多中间件本身，但内层 Trace 可能在 Panic 展开栈时先以成功状态结束，Logging 也可能来不及记录返回结果。

这不是所有系统唯一正确的顺序。

关键是每一层为什么在这里必须说得清楚。

---

# Trace 应该尽量覆盖完整请求

Trace ID 通常需要在请求刚进入系统时读取或创建。

然后向下游继续传递。

这样才能把下面的调用关联起来：

```text
网关
  ↓
订单服务
  ↓
库存服务
  ↓
数据库
```

如果中间某一层忘记传递上下文，后面的 Span 就可能变成一条新的链路。

因此 Trace Middleware 负责在入口提取上游 Trace，并创建包含当前 Span 的派生 Context。

但它只能完成第一步。

后面的每一层都必须继续传递这个 Context。

但业务 ID 仍然应该单独记录。

Trace 可能被采样，订单号和消息 ID 才能帮助定位具体业务。

---

# Context 传播不是 Middleware 一层完成的

Kratos Middleware 创建的 Context 应该沿着完整调用路径向下传递：

```text
Middleware
   ↓
Service
   ↓
Biz
   ↓
Data Repo
   ↓
数据库或下游 RPC
```

Service 不应该在进入 Biz 时重新创建 Context：

```go
func (s *OrderService) GetOrder(
	ctx context.Context,
	req *v1.GetOrderRequest,
) (*v1.GetOrderReply, error) {
	order, err := s.uc.Get(ctx, req.Id)
	if err != nil {
		return nil, err
	}
	return toReply(order), nil
}
```

Biz 调用 Repo 和其他用例时也继续使用同一个 `ctx`：

```go
func (uc *OrderUsecase) Get(
	ctx context.Context,
	id int64,
) (*Order, error) {
	return uc.orders.FindByID(ctx, id)
}
```

Data 调用数据库要使用 `QueryContext`、`ExecContext` 等接受 Context 的 API。

调用下游 Kratos Client 时，同样要传入当前业务 Context：

```go
func (r *paymentRepo) GetPayment(
	ctx context.Context,
	orderID int64,
) (*Payment, error) {
	reply, err := r.client.GetPayment(ctx, &paymentv1.GetPaymentRequest{
		OrderId: orderID,
	})
	if err != nil {
		return nil, mapPaymentError(err)
	}
	return toPayment(reply), nil
}
```

`tracing.Client()` 会从这个 `ctx` 读取当前 Span，并把 Trace Carrier 注入请求 Header。

如果这里换成 `context.Background()`，不仅会生成一条脱离当前请求的新链路，上游超时和取消也无法传到下游。

---

# Context Value 不会自动跨网络

Context Value 只存在于当前 Go 调用链。

例如：

- Data 放入 Context 的 `*sql.Tx` 只能在当前进程使用
- Middleware 放入的调用身份只在明确读取它的代码中存在
- 自定义 Request ID 不会因为放进 Context 就自动进入 HTTP Header

跨服务传播需要明确的载体：

- Trace 使用 OpenTelemetry Propagator 写入和提取 Carrier
- Kratos RPC Metadata 通过允许的 Header 传递
- MQ 使用消息 Header 保存 Trace 和业务关联标识

不能尝试序列化整个 Context，也不能把所有 Context Value 无脑发送给下游。

事务句柄、内部权限对象和本地缓存引用都不应该离开当前进程。

真正需要跨边界的数据应该有明确名称、稳定格式和安全范围。

---

# 超时中间件不能让工作自动停止

超时中间件可以在规定时间后向调用方返回错误。

但返回超时不代表内部工作一定停止。

如果业务代码、数据库驱动或下游调用没有响应取消信号，它们可能仍然继续执行。

于是出现：

```text
调用方看到超时
  ↓
调用方发起重试
  ↓
上一次请求仍在执行
```

所以超时控制需要和下面的能力配合：

- Context 取消传播
- 数据库查询超时
- 下游调用超时
- 幂等
- 并发控制

中间件只能发出取消信号。

不能保证所有代码都会正确响应。

---

# 认证和权限不是同一件事

认证回答：

> 你是谁？

权限回答：

> 你能不能执行这个操作？

认证通常适合放在公共中间件中，解析 Token 并写入身份上下文。

但具体业务权限可能依赖：

- 订单属于谁
- 当前用户是不是店铺管理员
- 资源当前处于什么状态

这部分不一定适合全部塞进通用中间件。

中间件可以提供身份和通用策略。

领域逻辑仍然需要保护和业务资源相关的权限规则。

---

# 不要把业务逻辑放进通用中间件

假设一个中间件开始处理：

- 创建订单幂等记录
- 检查库存
- 修改用户积分
- 决定订单能否退款

这些行为只属于少数接口，却被放进所有请求共用的执行链。

结果会变成：

- 中间件顺序难以调整
- 业务规则隐藏在入口之外
- 定时任务和 MQ 消费者无法复用
- 测试需要启动完整链路

通用中间件适合处理与传输和治理相关的横切能力。

业务规则应该留在应用或领域层。

---

# 错误转换应该保留语义

基础设施可能返回很多具体错误：

- 数据库唯一键冲突
- 连接超时
- 下游不可用
- 消息发送失败

对外接口不应该直接暴露驱动错误和内部堆栈。

但也不能把所有错误都转换成同一个“系统错误”。

错误转换需要保留调用方真正需要的语义：

- 是否是业务拒绝
- 是否可以重试
- 是否需要重新认证
- 是否属于资源冲突

同时，预期内的业务分支不应该随意作为系统 Error 穿过整条 Trace。

否则错误率和告警会失去意义。

这里需要注意 Kratos v3 的默认行为。

只要 Handler 最终返回的 `error` 不是 `nil`：

- Logging Middleware 就会按 Error 级别记录
- Tracing Middleware 就会执行 `RecordError`
- 当前 Span 会被标记为 Error

即使返回的是一个语义明确的 `Conflict` 或 `BadRequest`，它仍然是非 `nil` Error。

Kratos Error Code 和 Reason 可以帮助指标、日志和告警进一步分类，但不会自动阻止 Span 被标记为 Error。

因此已经被业务正常处理的分支，应该在处理位置收敛为正常返回。

确实需要向调用方表达失败的业务拒绝，则保留稳定 Error 语义，并在可观测平台按 Code 和 Reason 区分业务失败与系统故障。

---

# 公共中间件的故障半径很大

一个业务函数出错，通常只影响一个接口。

一个公共中间件出错，可能影响所有接口。

例如：

- Trace 上报阻塞请求
- 日志序列化发生 Panic
- 鉴权缓存不可用导致全部拒绝
- 限流配置错误导致正常请求被拦截

因此公共中间件需要更加保守：

- 明确超时
- 合理降级
- 避免无限阻塞
- 控制资源使用
- 提供独立开关
- 充分测试异常路径

复用越广的组件，变更风险越大。

---

# 默认配置也是架构决定

中间件通常拥有很多配置：

- 超时时间
- 采样率
- 限流阈值
- 是否记录请求体
- 哪些路径跳过认证

如果默认值不安全，每个服务都可能复制错误配置。

例如默认记录完整请求体，可能把密码和 Token 写入日志。

默认没有超时，可能让下游请求无限等待。

所以公共组件应该提供保守且可解释的默认值。

特殊服务可以覆盖，但需要明确知道自己改变了什么。

---

# HTTP 之外的入口也要治理

很多系统只给 HTTP 接口配置中间件。

但业务还可能从下面的入口执行：

- MQ 消费者
- 定时任务
- 命令行任务
- 内部 RPC

这些入口同样需要：

- Trace
- 超时
- 恢复
- 指标
- 身份或任务上下文

如果治理能力只能依赖 HTTP 中间件，其他入口就会变成观察盲区。

可以复用底层能力，但不同入口应该拥有适合自己的执行链。

---

# 请求 Context 不能直接交给后台任务

HTTP 或 gRPC 请求结束后，请求 Context 会被取消。

因此下面的代码并不可靠：

```go
func (uc *OrderUsecase) Create(ctx context.Context, order *Order) error {
	if err := uc.orders.Create(ctx, order); err != nil {
		return err
	}

	go uc.notifications.Send(ctx, order.ID)
	return nil
}
```

Handler 返回以后，后台 Goroutine 使用的 `ctx` 可能已经取消。

它也可能仍然携带已经提交的事务句柄、当前请求身份和即将结束的 Span。

不要简单使用 `context.WithoutCancel(ctx)` 解决这个问题。

它虽然去掉了 Deadline 和 Cancellation，却仍然保留父 Context 的 Value，可能把已经失效的事务句柄继续带下去。

必须可靠完成的工作应该进入持久化任务或 MQ。

真正允许丢失的进程内异步工作，也应该创建独立且有明确超时的 Context，并只显式复制它需要的业务标识。

---

# MQ 需要重新建立一次消费 Context

Go 的 `context.Context` 不能直接写进消息。

生产者应该把 Trace Carrier、消息 ID 和必要业务 ID 写入消息 Header。

消费者收到消息后，以消费者自己的生命周期 Context 为基础，提取 Trace 信息并创建新的 Consumer Span：

```go
func (c *OrderConsumer) Handle(
	ctx context.Context,
	msg Message,
) error {
	ctx = c.propagator.Extract(ctx, headerCarrier(msg.Headers))
	ctx, span := c.tracer.Start(
		ctx,
		"order.created",
		trace.WithSpanKind(trace.SpanKindConsumer),
	)
	defer span.End()

	ctx, cancel := context.WithTimeout(ctx, 10*time.Second)
	defer cancel()

	return c.uc.HandleCreated(ctx, msg.OrderID, msg.ID)
}
```

每一次投递都拥有自己的消费执行过程。

消息 ID 和幂等键负责识别是不是同一件业务工作，Trace 负责描述这一次实际经过了哪些处理节点。

定时任务和命令行任务也一样：它们没有入站 HTTP Context，需要由入口创建自己的根 Context、Deadline 和 Trace。

---

# Kratos 为什么把自己设计成工具箱

Kratos 不要求项目必须使用某一种 ORM、注册中心或配置中心。

框架核心提供的是：

- HTTP 和 gRPC Transport
- Middleware 抽象
- 服务生命周期
- 统一 Error 和 Metadata
- API 代码生成
- 可插拔的注册、配置和可观测组件

这和前面讲的架构原则一致。

框架定义稳定的连接位置，具体基础设施通过组件接入。

项目可以使用默认 Layout，也可以根据业务规模调整结构。

但无论怎样调整，都应该保留清楚的依赖方向。

---

# 在 Server 中明确组装 Middleware

Kratos 的 HTTP 和 gRPC Server 使用同一套 `middleware.Middleware` 抽象。

可以在 `internal/server` 中集中组装：

```go
// internal/server/http.go
package server

import (
	"log/slog"
	"time"

	v1 "example/api/order/v1"
	"example/internal/service"

	"github.com/go-kratos/kratos/contrib/otel/v3/tracing"
	"github.com/go-kratos/kratos/v3/middleware/logging"
	"github.com/go-kratos/kratos/v3/middleware/ratelimit"
	"github.com/go-kratos/kratos/v3/middleware/recovery"
	"github.com/go-kratos/kratos/v3/middleware/validate"
	"github.com/go-kratos/kratos/v3/transport/http"
)

func NewHTTPServer(
	svc *service.OrderService,
	logger *slog.Logger,
) *http.Server {
	srv := http.NewServer(
		http.Address(":8000"),
		http.Timeout(2*time.Second),
		http.Middleware(
			tracing.Server(),
			logging.Server(logger),
			recovery.Recovery(),
			ratelimit.Server(),
			validate.Validator(),
		),
	)
	v1.RegisterOrderServiceHTTPServer(srv, svc)
	return srv
}
```

这里把 `tracing.Server()` 放在 `logging.Server()` 之前。

这样 Logging Middleware 执行时，上下文里已经存在当前 Span。

Recovery 放在 Logging 里面，是为了先把业务 Panic 转成 Error，让外层 Logging 和 Trace 能够记录失败结果。

这意味着 Recovery 不负责保护它外面的 Trace 和 Logging 自身。

公共可观测中间件必须保持简单、经过充分测试；如果项目必须连这些中间件的 Panic 一起恢复，可以在 Transport 最外层再提供一个只负责兜底的恢复层，但不能因此忽略业务 Panic 的 Trace 状态。

这个顺序仍然要根据项目需求调整，但不能在不同服务里随意排列。

---

# 注册 Trace Middleware 不等于已经上报 Trace

`tracing.Server()` 和 `tracing.Client()` 负责创建 Span、提取和注入 Trace Carrier。

它们仍然需要一个真正工作的 OpenTelemetry `TracerProvider`。

应用应该在创建 HTTP Server、gRPC Server 和 Client 之前完成初始化：

```go
func NewTracerProvider(
	exporter sdktrace.SpanExporter,
) *sdktrace.TracerProvider {
	resource := resource.NewSchemaless(
		attribute.String("service.name", "order-service"),
	)

	tp := sdktrace.NewTracerProvider(
		sdktrace.WithBatcher(exporter),
		sdktrace.WithResource(resource),
		sdktrace.WithSampler(
			sdktrace.ParentBased(
				sdktrace.TraceIDRatioBased(0.1),
			),
		),
	)
	otel.SetTracerProvider(tp)
	return tp
}
```

如果没有设置 Provider，默认 Provider 可能只返回不记录、不导出的 Span。

应用退出时还要使用一个独立、带超时的 Context 调用 `tp.Shutdown(ctx)`，让批量缓存中的 Span 有机会完成导出。

不能使用已经取消的请求 Context 执行关闭操作。

采样率也不是简单设成固定百分比。

通常应该使用 `ParentBased`：上游已经决定采样时，下游继续保持同一个决定；没有上游 Span 时，再由本服务决定是否采样。

---

# 传递 Context 不会自动生成所有 Span

Server Middleware 会生成服务端 Span，Client Middleware 会生成下游 RPC Span。

但普通的 Biz 方法和 Repo 方法不会因为接收了 `ctx` 就自动拥有独立 Span。

要观察数据库调用，需要使用支持 OpenTelemetry 的数据库 Instrumentation，或者在真正有观测价值的业务步骤手动创建 Span。

不要为每一个小函数都创建 Span。

Span 应该对应能够帮助定位延迟和故障的边界，例如：

- 一次下游 RPC
- 一条重要 SQL 或一次事务
- 一次外部支付调用
- 一段耗时明显的业务阶段

Context 负责把父 Span 带到这些位置，Instrumentation 才负责创建新的子 Span。

---

# Context 里有 Span 不等于日志里有 Trace ID

`tracing.Server()` 会把当前 Span 放进派生 Context。

`logging.Server(logger)` 也会把这个 Context 传给 `slog.Logger`。

但标准库的 `slog.JSONHandler` 和 `slog.TextHandler` 不会自动把 Span 转成日志字段。

要关联日志和 Trace，还需要满足下面一种方式：

- 使用 Kratos contrib 提供的 OpenTelemetry slog Handler
- 自定义 slog Handler，从 Context 提取 Span
- 记录日志时显式追加 `tracing.TraceAttrs(ctx)`

例如创建 OpenTelemetry Logger：

```go
import (
	otellog "github.com/go-kratos/kratos/contrib/otel/v3/log"
	kratoslog "github.com/go-kratos/kratos/v3/log"
)

logger := kratoslog.NewLogger(
	otellog.NewHandler("order-service"),
)
```

这种方式的前提是项目同时配置了 OpenTelemetry LoggerProvider 和日志 Exporter。

如果日志仍然输出到标准输出或文件，就应该为现有 slog Handler 增加 Trace 关联能力，或者像后面的示例一样显式追加 `tracing.TraceAttrs(ctx)`。

这个 Handler 会从日志调用传入的 Context 中增加 `trace_id`、`span_id` 和 `trace_flags`。

所以业务代码记录请求日志时也应该使用 `InfoContext`、`LogAttrs` 等接收 Context 的方法。

只调用不带 Context 的 `logger.Info`，日志 Handler 就无法取得当前 Span。

---

# 自定义 Middleware 应该保持通用

Kratos Middleware 的本质是一个包装 Handler 的函数：

```go
package middlewarex

import (
	"context"
	"log/slog"
	"time"

	"github.com/go-kratos/kratos/contrib/otel/v3/tracing"
	"github.com/go-kratos/kratos/v3/middleware"
	"github.com/go-kratos/kratos/v3/transport"
)

func Timing(logger *slog.Logger) middleware.Middleware {
	return func(next middleware.Handler) middleware.Handler {
		return func(ctx context.Context, req any) (any, error) {
			startedAt := time.Now()
			operation := "unknown"
			if tr, ok := transport.FromServerContext(ctx); ok {
				operation = tr.Operation()
			}

			reply, err := next(ctx, req)
			attrs := []slog.Attr{
				slog.String("operation", operation),
				slog.Duration("latency", time.Since(startedAt)),
			}
			attrs = append(attrs, tracing.TraceAttrs(ctx)...)
			level := slog.LevelInfo
			if err != nil {
				level = slog.LevelError
				attrs = append(attrs, slog.Any("error", err))
			}
			logger.LogAttrs(ctx, level, "request completed", attrs...)
			return reply, err
		}
	}
}
```

这个中间件只依赖统一的 Operation 和 Context。

因此 HTTP 和 gRPC 都可以使用。

Kratos 已经提供了 Logging Middleware，上面的代码主要用于展示自定义 Middleware 的结构。

真实项目不应该再注册一个功能重复的请求日志中间件，否则同一次请求会产生两份记录。

如果代码开始根据某个订单状态执行退款，它就已经不再是通用 Middleware，而是业务用例的一部分。

> 示例使用 Kratos v3。v3 日志基于标准库 `log/slog`；v2 项目的 Logging API 和导入路径有所不同。

---

# 小结

架构设计不是多创建几层目录。

它需要控制：

- 依赖方向
- 业务边界
- 技术细节的影响范围
- 公共能力的故障半径

中间件则把 Trace、认证、限流、超时等横切能力放进统一执行链。

设计中间件时需要关注：

- 执行顺序
- 上下文传播
- 错误语义
- 取消是否真正生效
- 是否错误地混入业务规则
- 公共组件失败时影响多大

好的架构不会让系统看起来最复杂。

它真正带来的效果是：

> **业务变化时只修改应该修改的地方，基础设施故障时不会轻易拖垮整个系统。**
