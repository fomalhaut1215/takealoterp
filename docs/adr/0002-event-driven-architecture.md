# ADR 0002: 事件驱动架构用于服务间异步通信

## 状态
已接受 (Accepted)

## 日期
2026-09-30

## 背景

在微服务架构下，服务之间需要通信以完成业务流程。例如：
- 订单创建后需要触发库存扣减
- 库存低于阈值需要触发补货建议
- AI 生成的产品描述需要更新到 Product Service
- 订单状态变更需要通知 Analytics Service 更新报表

同步 REST/gRPC 调用存在问题：
1. **紧耦合**: 调用方需要知道被调用方的地址和接口
2. **级联故障**: 下游服务不可用导致上游服务阻塞
3. **性能瓶颈**: 长链路调用导致响应变慢
4. **扩展性差**: 新增订阅者需要修改发布者代码

## 决策

采用 **事件驱动架构 (Event-Driven Architecture)** 进行服务间异步通信：

### 架构模式
使用 **发布-订阅 (Pub/Sub)** 模式，通过消息队列解耦服务。

### 技术选型
- **消息队列**: **RabbitMQ**（优先选择）
  - 成熟稳定，运维简单
  - 支持多种消息模式（Fanout、Topic、Direct）
  - 良好的死信队列和重试机制
  
- **备选方案**: Kafka（若未来数据量极大或需要事件溯源时切换）

### 事件分类

#### 1. 领域事件 (Domain Events)
业务状态变更触发的事件：
- `OrderCreated`: 订单创建
- `OrderShipped`: 订单已发货
- `OfferPriceChanged`: 报价价格变更
- `InventoryLowStock`: 库存低于阈值
- `ReturnRequested`: 退货请求

#### 2. 集成事件 (Integration Events)
跨服务协作的事件：
- `AIDescriptionGenerated`: AI 生成的描述已完成
- `DemandForecastUpdated`: 需求预测已更新
- `RepricingSuggestionCreated`: 调价建议已生成

#### 3. 命令事件 (Command Events)
触发特定动作的事件（谨慎使用，避免隐式命令）：
- `SyncInventoryToTakealot`: 同步库存到 Takealot

### 事件结构标准
```json
{
  "eventId": "uuid",
  "eventType": "OrderCreated",
  "timestamp": "2026-09-30T10:00:00Z",
  "source": "order-service",
  "version": "1.0",
  "data": {
    // 领域特定数据
  },
  "metadata": {
    "correlationId": "trace-uuid",
    "userId": "seller-123"
  }
}
```

### 服务间事件流示例

#### 场景 1: 订单创建流程
```
OrderService → [OrderCreated] 
  ├─> InventoryService: 扣减库存
  ├─> AnalyticsService: 更新销售数据
  └─> CustomerService: 发送订单确认消息
```

#### 场景 2: AI 产品描述生成
```
ProductService → [DescriptionGenerationRequested]
  └─> AIService: 生成描述
      └─> [AIDescriptionGenerated]
          └─> ProductService: 更新 Offer
```

#### 场景 3: 库存预警补货
```
InventoryService → [InventoryLowStock]
  ├─> AIService: 触发需求预测
  │   └─> [DemandForecastUpdated]
  │       └─> ProductService: 生成补货建议
  └─> CustomerService: 通知卖家
```

## 后果

### 优势
✅ **松耦合**: 服务不直接依赖彼此，仅依赖事件契约
✅ **弹性**: 下游服务短暂不可用不影响上游
✅ **可扩展**: 新增订阅者无需修改发布者
✅ **审计追踪**: 所有事件可记录用于审计和重放
✅ **最终一致性**: 自然支持分布式系统的最终一致性

### 劣势
❌ **复杂度**: 调试困难，需要事件追踪工具
❌ **消息顺序**: 需要处理乱序和重复消费
❌ **事务边界**: 跨服务事务需要 Saga 模式
❌ **运维负担**: 需要维护消息队列集群

### 缓解措施
1. **幂等性设计**: 所有事件消费者必须幂等处理
2. **事件版本化**: 事件结构变更通过版本号向后兼容
3. **死信队列**: 处理失败的消息进入 DLQ，人工介入
4. **链路追踪**: 使用 correlationId 串联整个调用链
5. **事件存储**: 关键事件持久化到 Event Store 用于审计

## 同步 vs 异步决策矩阵

| 场景 | 通信方式 | 原因 |
|------|---------|------|
| 用户查询 Offer 详情 | 同步 (REST/gRPC) | 需要立即响应 |
| 订单创建后扣减库存 | 异步 (Event) | 允许最终一致性 |
| AI 生成产品描述 | 异步 (Event) | 耗时操作 |
| 调用 Takealot API 创建 Offer | 同步 (REST) | 需要立即获取结果 |
| 更新销售报表 | 异步 (Event) | 非实时需求 |
| 权限验证 | 同步 (gRPC) | 低延迟要求 |

## 替代方案

### 方案 A: 纯同步通信 (REST/gRPC)
- **优势**: 简单直观，调试容易
- **劣势**: 紧耦合，级联故障风险高
- **为何拒绝**: 不符合微服务松耦合原则

### 方案 B: 数据库共享
- **优势**: 无消息队列复杂度
- **劣势**: 数据库成为耦合点，违反微服务独立数据库原则
- **为何拒绝**: 失去微服务核心优势

### 方案 C: HTTP Webhook
- **优势**: 无需消息队列
- **劣势**: 需要服务暴露公网端点，重试机制复杂
- **为何拒绝**: 不适合内部服务通信

## 相关决策
- ADR 0001: 采用微服务架构
- ADR 0004: Saga 模式处理分布式事务（待定）
- ADR 0005: 事件溯源用于审计追踪（待定）

## 参考资料
- [Event-Driven Architecture Pattern](https://microservices.io/patterns/data/event-driven-architecture.html)
- [RabbitMQ vs Kafka](https://www.confluent.io/blog/kafka-vs-rabbitmq/)
- [Building Event-Driven Microservices](https://www.oreilly.com/library/view/building-event-driven-microservices/9781492057888/)
