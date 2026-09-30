# Takealot ERP 系统 - 领域上下文

## 领域概述

Takealot ERP 是一个面向南非 Takealot 电商平台的智能运营管理系统，旨在通过 AI 深度集成和自动化技术，帮助本地商户降低运营门槛、提升效率并增加利润。

---

## 核心概念

### 商品管理 (Product Management)

**Offer (报价)**
- Takealot 平台上的产品销售实例
- 包含价格、库存、配送选项等信息
- 一个产品可能有多个 Seller 的 Offer

**Batch Operation (批量操作)**
- 通过 Takealot API 批量创建/更新 Offer
- 用于规模化商品上架和价格调整

**Buy Box (购物车按钮)**
- Takealot 产品页面上的主购买入口
- 多个 Seller 竞争同一产品的 Buy Box 展示
- 由价格、库存、履约表现等因素决定

**SKU (Stock Keeping Unit)**
- 库存保有单位，唯一标识一个可销售商品

### 订单与履约 (Order & Fulfillment)

**Order Routing (订单路由)**
- 将订单分配到最优仓库或 Dropshipping 供应商
- 考虑因素：库存位置、配送时效、成本

**Order Split (订单拆分)**
- 将单个订单拆分到多个仓库履约
- 优化配送时效和成本

**Leadtime Stock (备货周期库存)**
- 从供应商处理到入 Takealot 仓库的在途库存

**Warehouse Stock (仓库库存)**
- 已在 Takealot 配送中心的可售库存

### 托管服务 (Managed Service)

**半托管模式 (Semi-Managed)**
- 卖家保留最终决策权
- 平台提供自动化建议和执行能力
- 适合有一定经验但希望提升效率的卖家

**全托管模式 (Fully-Managed)**
- 平台全权负责运营决策和执行
- 卖家仅提供产品供应链
- 按销售额分成或固定服务费
- 适合完全不熟悉电商的商户

**Dropshipping (代发货)**
- 零库存模式，订单直接由供应商发货
- 降低资金占用和仓储风险

### AI 能力 (AI Capabilities)

**AI Product Description Generation (AI 产品描述生成)**
- 自动生成优化的产品标题和描述
- 多语言支持（英语、南非荷兰语）
- SEO 优化

**AI Repricing (AI 智能调价)**
- 基于竞品价格、库存、Buy Box 状态动态调价
- 在保证利润前提下最大化销量

**Demand Forecasting (需求预测)**
- 基于历史销售和外部信号预测未来需求
- 驱动库存优化和补货决策

**AI Customer Service (AI 客服)**
- 多渠道自动响应（WhatsApp、邮件、SMS）
- 自动处理退货工作流

### 数据分析 (Analytics)

**True Profit (真实利润)**
- 扣除 Takealot 所有费用后的净利润
- 费用包括：月费、佣金、履约费、仓储费

**Inventory Health (库存健康度)**
- 评估库存周转率、滞销风险、缺货风险

**Sales Velocity (销售速度)**
- 单位时间内的销售量，用于需求预测

---

## 边界与约束

### 包含在系统内
- Takealot 平台商品和订单管理
- AI 驱动的运营自动化
- 数据分析与决策支持
- 托管服务运营

### 不包含
- Takealot 平台本身的功能（由 Takealot 提供）
- 供应商管理系统（假设供应商关系由卖家自行维护）
- 会计和税务申报（提供数据导出，但不做申报）

### 集成边界
- **Takealot API**: 通过官方 Seller API 集成
- **物流服务商**: ShipBob、本地快递公司
- **Dropshipping 平台**: Syncee、Dropstore
- **支付网关**: 南非本地支付方式
- **AI 服务**: OpenAI/Anthropic Claude API

---

## 业务规则

### 定价规则
1. 最低价格不得低于成本价 + 最低利润率
2. 调价频率限制：每个 SKU 每天最多调整 3 次
3. Buy Box 争夺：价格相差 5% 以内考虑其他因素（库存、评分）

### 库存规则
1. 安全库存：至少保持 7 天销售量
2. 缺货预警：库存低于 3 天销售量时触发
3. 滞销标准：60 天无销售视为滞销

### 订单路由规则
1. 优先选择距离客户最近的仓库
2. 库存不足时触发 Dropshipping
3. 高价值订单（>R1000）优先发 Takealot FBA 以确保时效

### 退货规则
1. 符合 Takealot 退货政策（通常 30 天内）
2. 未开封商品全额退款
3. 已使用商品扣除折旧费用

---

## 避免的术语

❌ **不要使用**:
- "Product" (产品) — 使用 "Offer" (报价) 指代 Takealot 上的销售实例
- "Inventory" 单独使用 — 明确是 "Warehouse Stock" 还是 "Leadtime Stock"
- "Managed" 单独使用 — 明确是 "Semi-Managed" 还是 "Fully-Managed"

✅ **推荐使用**:
- "Offer" 而非 "Product Listing"
- "Batch Operation" 而非 "Bulk Upload"
- "Order Routing" 而非 "Order Allocation"
- "True Profit" 而非 "Net Profit"

---

## 关键利益相关者

### 卖家 (Seller)
- 在 Takealot 上销售商品的商户
- 可能是个人、小型企业或品牌

### 平台运营团队 (Platform Operations)
- 负责托管服务的选品、上架、客服等

### 供应商 (Supplier)
- Dropshipping 模式下的货源提供方

### 最终消费者 (End Customer)
- 在 Takealot 上购买商品的用户

---

## 成功指标

### 卖家端
- **运营效率**: 商品上架时间从 2 小时降至 10 分钟
- **利润提升**: 通过智能调价和库存优化提升 15% 利润
- **时间节省**: 每周节省 10+ 小时手动运营时间

### 平台端
- **托管服务满意度**: NPS > 50
- **自动化率**: 80% 订单无需人工介入
- **AI 准确性**: 产品描述采用率 > 85%, 需求预测 MAPE < 20%

---

## 参考文档

详细调研报告见: GitHub Issue #1
