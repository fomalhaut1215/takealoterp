# ADR 0004: 后端技术栈选择 - Node.js + NestJS

## 状态
已接受 (Accepted)

## 日期
2026-09-30

## 背景

需要为 Takealot ERP 的微服务选择后端技术栈，考虑因素：
- 开发效率和生产力
- 与前端技术栈的协同（类型共享）
- 微服务架构支持
- AI 集成能力（调用 OpenAI/Claude API）
- 异步 I/O 性能（大量外部 API 调用）
- 团队技能和招聘难度
- 生态系统成熟度

系统特点：
- I/O 密集型（Takealot API、AI API、数据库查询）
- 需要大量 HTTP/REST 调用
- 实时数据处理（订单同步、库存更新）
- 复杂的业务逻辑和工作流

## 决策

### 主技术栈: Node.js + NestJS + TypeScript

**核心技术:**
- **运行时**: Node.js 20 LTS
- **框架**: NestJS 10+ (企业级 Node.js 框架)
- **语言**: TypeScript 5+
- **API 风格**: RESTful (主) + GraphQL (Analytics Service)
- **ORM**: Prisma (类型安全的现代 ORM)
- **验证**: class-validator + class-transformer
- **文档**: Swagger/OpenAPI 自动生成

**为何选择 NestJS:**
1. **架构清晰**: 模块化设计，天然支持微服务
2. **TypeScript 优先**: 与前端共享类型定义
3. **依赖注入**: 易于测试和扩展
4. **装饰器语法**: 简洁优雅的路由和验证
5. **内置支持**: WebSocket、gRPC、消息队列开箱即用
6. **丰富生态**: 大量官方和社区模块

### 微服务通信

**同步通信:**
- **内部服务间**: gRPC (高性能二进制协议)
- **客户端调用**: REST API (JSON)

**异步通信:**
- **消息队列**: RabbitMQ (通过 @nestjs/microservices)
- **事件总线**: NestJS EventEmitter (服务内事件)

### 数据库层

**主数据库:**
- **PostgreSQL 15+**
  - 成熟可靠，ACID 保证
  - JSON 支持（灵活存储 Takealot API 响应）
  - 丰富的扩展（PostGIS、全文搜索）
  
**ORM:**
- **Prisma**
  - 类型安全的查询（编译时检查）
  - 自动生成 TypeScript 类型
  - Migration 管理简单
  - Prisma Studio（可视化数据库管理）

**缓存:**
- **Redis**
  - 会话存储
  - API 响应缓存（Takealot 数据）
  - 分布式锁（防止重复调价）
  - 消息队列补充

**时序数据:**
- **TimescaleDB** (PostgreSQL 扩展)
  - 销售数据、库存变化时序分析
  - 与 PostgreSQL 无缝集成
  - SQL 查询，学习成本低

### 项目结构（NestJS 标准）

```
services/
├── api-gateway/           # API 网关
│   ├── src/
│   │   ├── auth/          # 认证模块
│   │   ├── common/        # 通用模块
│   │   └── main.ts
│   └── package.json
│
├── product-service/       # 商品服务
│   ├── src/
│   │   ├── offers/        # Offer CRUD
│   │   ├── inventory/     # 库存管理
│   │   ├── takealot/      # Takealot API 集成
│   │   ├── prisma/        # Prisma schema
│   │   └── main.ts
│   └── package.json
│
├── order-service/         # 订单服务
├── ai-service/            # AI 服务
├── analytics-service/     # 数据分析服务
├── customer-service/      # 客服服务
├── user-service/          # 用户服务
└── shared/                # 共享类型和工具
    ├── types/             # TypeScript 类型
    ├── events/            # 事件定义
    └── utils/             # 工具函数
```

### AI Service 特殊处理

**为何不用 Python:**
虽然 Python 在 AI/ML 领域更流行，但：
1. **AI 服务主要调用第三方 API**（OpenAI/Claude），非自建模型
2. **统一技术栈**降低运维复杂度
3. **TypeScript SDK 成熟**（@anthropic-ai/sdk, openai）
4. **异步 I/O 优势**：Node.js 处理大量并发 API 调用更高效

**需求预测模型:**
- 初期使用第三方 API（如 AWS Forecast）
- 若自建模型，可独立部署 Python 服务（FastAPI）并通过 gRPC 调用

## 后果

### 优势
✅ **全栈 TypeScript**: 前后端类型共享，减少接口对齐成本
✅ **开发效率**: NestJS CLI 快速生成代码，装饰器简化逻辑
✅ **异步性能**: Node.js 非阻塞 I/O 适合大量 API 调用场景
✅ **生态丰富**: npm 包数量最多，第三方集成简单
✅ **团队协同**: 前端开发者可轻松切换到后端开发
✅ **AI 友好**: Claude Code 对 NestJS + TypeScript 支持极佳

### 劣势
❌ **CPU 密集型弱**: 单线程，不适合复杂计算（但可 Worker Threads）
❌ **运行时错误**: 虽然 TypeScript 编译检查，但运行时仍可能出错
❌ **内存占用**: Node.js 内存占用比 Go/Rust 高

### 缓解措施
1. **CPU 密集任务**: 使用 Worker Threads 或独立 Python 服务
2. **运行时验证**: class-validator 在入口处验证所有输入
3. **内存优化**: 使用流式处理大数据，定期监控内存泄漏
4. **性能监控**: 接入 New Relic / Datadog APM

## 技术选型对比

### 后端框架对比

| 框架 | 语言 | 优势 | 劣势 | 决策 |
|------|------|------|------|------|
| **NestJS** | TypeScript | 架构清晰，企业级，全栈TS | 学习曲线 | ✅ **选择** |
| Express.js | JavaScript | 简单灵活 | 无架构约束，难以维护 | ❌ |
| Fastify | JavaScript | 性能最佳 | 生态不如Express，架构弱 | ❌ |
| Go + Gin | Go | 性能强，并发好 | 类型不共享，团队需学新语言 | ❌ |
| Spring Boot | Java | 成熟企业级 | 过于重量，启动慢 | ❌ |
| FastAPI | Python | 简洁，AI生态好 | 异步I/O不如Node，类型不共享 | ❌ |

### ORM 对比

| ORM | 优势 | 劣势 | 决策 |
|-----|------|------|------|
| **Prisma** | 类型安全，生成TypeScript类型 | 较新，社区小 | ✅ **选择** |
| TypeORM | NestJS官方推荐，成熟 | 类型推断弱，装饰器繁琐 | ❌ |
| Sequelize | 最成熟 | 无TypeScript优先，API老旧 | ❌ |
| Knex.js | SQL构建器，灵活 | 无ORM便利，手写SQL多 | ❌ |

**理由**: Prisma 的类型安全是刚需，避免运行时数据库错误。

## 性能目标

### API 响应时间
- **简单查询**: < 100ms (P95)
- **复杂查询**: < 500ms (P95)
- **AI 生成描述**: < 5s (P95)

### 吞吐量
- **API Gateway**: 10,000 req/s (单实例)
- **Product Service**: 5,000 req/s
- **Order Service**: 3,000 req/s

### 数据库连接池
- **连接数**: 每服务 20-50 个连接
- **超时**: 30s

## 开发体验优化

### 开发工具
- **NestJS CLI**: 快速生成模块/控制器/服务
- **Prisma Studio**: 可视化数据库管理
- **热重载**: ts-node-dev (开发环境秒级重启)
- **调试**: VS Code 断点调试支持

### 测试策略
- **单元测试**: Jest (NestJS 默认)
- **集成测试**: Supertest (HTTP 测试)
- **E2E 测试**: 使用 Docker Compose 启动依赖服务
- **测试覆盖率**: > 80%

### 代码质量
- **Linter**: ESLint + @typescript-eslint
- **Formatter**: Prettier
- **Git Hooks**: Husky + lint-staged
- **CI/CD**: GitHub Actions (测试 + 构建 + 部署)

## 部署策略

### 容器化
- **基础镜像**: node:20-alpine (体积最小)
- **多阶段构建**: 编译 + 生产分离
- **镜像大小**: < 150MB (含依赖)

### 编排
- **Kubernetes**: 生产环境
- **Docker Compose**: 本地开发

### 监控
- **日志**: Winston + Elasticsearch + Kibana
- **指标**: Prometheus + Grafana
- **追踪**: Jaeger (OpenTelemetry)
- **健康检查**: Kubernetes liveness/readiness probes

## 替代方案

### 方案 A: Go + Gin/Echo
- **优势**: 性能强，编译型语言，并发原生支持
- **劣势**: 类型不共享，团队学习成本高，AI集成库不如Node.js
- **为何拒绝**: 开发效率低于TypeScript，类型共享优势丧失

### 方案 B: Python + FastAPI
- **优势**: AI生态好，代码简洁
- **劣势**: 异步I/O不如Node.js，类型不共享，性能一般
- **为何拒绝**: ERP系统I/O密集，Node.js更合适；AI功能主要调API非自建模型

### 方案 C: Java + Spring Boot
- **优势**: 企业级成熟，大量现成轮子
- **劣势**: 启动慢，内存占用大，开发繁琐
- **为何拒绝**: 过度工程，不适合快速迭代的初创项目

## 相关决策
- ADR 0001: 采用微服务架构
- ADR 0002: 事件驱动架构
- ADR 0003: 前端技术栈 React + TypeScript (类型共享)
- ADR 0005: 使用 Prisma 作为 ORM（待整合）

## 参考资料
- [NestJS 官方文档](https://docs.nestjs.com/)
- [Prisma 最佳实践](https://www.prisma.io/docs/guides/performance-and-optimization)
- [Node.js 性能优化](https://nodejs.org/en/docs/guides/simple-profiling/)
- [TypeScript 全栈开发](https://www.typescriptlang.org/docs/handbook/intro.html)
