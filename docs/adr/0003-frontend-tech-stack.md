# ADR 0003: 前端技术栈选择 - React + TypeScript

## 状态
已接受 (Accepted)

## 日期
2026-09-30

## 背景

Takealot ERP 需要两个前端应用：
1. **Web Dashboard**: 桌面端卖家管理平台
2. **Mobile App**: 移动端轻量管理工具（参考 TSeller 的移动端优势）

需要考虑的因素：
- 开发团队技能储备
- 组件生态和 UI 库
- TypeScript 支持（类型安全）
- 跨平台能力（Web + Mobile）
- 长期维护成本
- AI 辅助开发友好度

## 决策

### Web Dashboard: React + TypeScript + Vite

**核心技术栈:**
- **框架**: React 18+ (Hooks + Context)
- **语言**: TypeScript 5+
- **构建工具**: Vite（快速开发体验）
- **状态管理**: Zustand（轻量级，避免 Redux 复杂度）
- **路由**: React Router v6
- **UI 组件库**: **Ant Design** (antd)
  - 企业级中后台 UI 组件库
  - 开箱即用的表格、表单、图表组件
  - 完善的 TypeScript 支持
  - 成熟的国际化方案
- **数据可视化**: Apache ECharts (via echarts-for-react)
- **HTTP 客户端**: Axios + React Query (数据缓存和自动重试)
- **表单管理**: React Hook Form
- **样式方案**: CSS Modules + Tailwind CSS（实用优先）

**项目结构:**
```
web/
├── src/
│   ├── components/     # 通用组件
│   ├── features/       # 功能模块
│   │   ├── products/
│   │   ├── orders/
│   │   ├── analytics/
│   │   └── settings/
│   ├── hooks/          # 自定义 Hooks
│   ├── services/       # API 调用
│   ├── stores/         # Zustand stores
│   ├── types/          # TypeScript 类型定义
│   └── utils/          # 工具函数
├── package.json
└── vite.config.ts
```

### Mobile App: React Native + TypeScript

**核心技术栈:**
- **框架**: React Native 0.73+
- **语言**: TypeScript 5+
- **导航**: React Navigation
- **状态管理**: Zustand（与 Web 保持一致）
- **UI 组件库**: React Native Paper（Material Design）
- **数据请求**: React Query（与 Web 保持一致）

**为何不选择 Flutter:**
- 团队已有 React 经验，学习成本低
- 与 Web 共享业务逻辑代码
- AI 辅助开发（Claude/GitHub Copilot）对 React 生态支持更好

## 后果

### 优势
✅ **开发效率**: Vite 秒级启动，HMR 快速反馈
✅ **类型安全**: TypeScript 减少运行时错误
✅ **组件复用**: Web 和 Mobile 共享业务逻辑 Hooks
✅ **生态成熟**: React 生态庞大，第三方库丰富
✅ **AI 友好**: Claude Code 和 Copilot 对 React + TS 支持极佳
✅ **团队协作**: 统一技术栈，降低上下文切换

### 劣势
❌ **包体积**: React 运行时有一定体积（但 Vite Tree-shaking 优化良好）
❌ **学习曲线**: Hooks + TypeScript 对新手有门槛
❌ **React Native 性能**: 复杂动画和列表性能不如原生

### 缓解措施
1. **代码分割**: React.lazy + Suspense 按需加载页面
2. **性能优化**: 使用 React.memo、useMemo、useCallback 避免无效渲染
3. **打包优化**: Vite 自动 Tree-shaking，移除未使用代码
4. **Mobile 性能**: 使用 React Native FlatList 优化长列表

## 技术决策细节

### 为何选择 Ant Design 而非其他 UI 库？

| UI 库 | 优势 | 劣势 | 决策 |
|-------|------|------|------|
| **Ant Design** | 企业级组件丰富，中后台场景完善 | 包体积较大 | ✅ **选择** |
| Material-UI | Material Design 风格统一 | 定制复杂，性能一般 | ❌ |
| Chakra UI | 轻量，样式灵活 | 组件数量少，表格较弱 | ❌ |
| Shadcn UI | 现代化，Tailwind 集成好 | 组件需手动复制，维护成本高 | ❌ |

**理由**: ERP 系统需要大量表格、表单、数据展示组件，Ant Design 开箱即用，减少造轮子时间。

### 为何选择 Zustand 而非 Redux？

- **Zustand 优势**:
  - API 简洁，无 boilerplate
  - TypeScript 支持优秀
  - 无需 Provider 包裹
  - 体积小（2KB vs Redux 8KB）
  
- **Redux 劣势**:
  - Action/Reducer 样板代码多
  - 学习曲线陡峭
  - 对于中小型应用过度设计

**理由**: ERP 系统状态管理需求中等，Zustand 的简洁性足够，且与 React Query 配合良好（服务端状态交给 React Query）。

### 为何选择 React Query？

- 自动缓存和重试
- 自动处理 loading/error 状态
- 乐观更新和后台同步
- 减少 50% 状态管理代码

**示例:**
```typescript
// 传统方式
const [data, setData] = useState(null)
const [loading, setLoading] = useState(false)
const [error, setError] = useState(null)

useEffect(() => {
  setLoading(true)
  fetch('/api/offers')
    .then(res => res.json())
    .then(setData)
    .catch(setError)
    .finally(() => setLoading(false))
}, [])

// React Query 方式
const { data, isLoading, error } = useQuery({
  queryKey: ['offers'],
  queryFn: fetchOffers
})
```

## 替代方案

### 方案 A: Vue 3 + TypeScript
- **优势**: 更简洁的模板语法，性能略优
- **劣势**: 生态不如 React 丰富，移动端选择少（Vue Native 不成熟）
- **为何拒绝**: 无法与 React Native 共享代码

### 方案 B: Next.js (React 全栈框架)
- **优势**: SSR/SSG 支持，SEO 友好
- **劣势**: ERP 是认证后台，不需要 SEO；复杂度增加
- **为何拒绝**: 过度工程，SPA 足够

### 方案 C: Svelte + SvelteKit
- **优势**: 无虚拟 DOM，运行时小，性能好
- **劣势**: 生态小，招聘困难，AI 辅助支持弱
- **为何拒绝**: 技术风险高，团队不熟悉

## 性能目标

### Web Dashboard
- **首屏加载**: < 2s (3G 网络)
- **Lighthouse 评分**: Performance > 90
- **包体积**: Gzipped < 500KB (不含 vendor)

### Mobile App
- **启动时间**: < 3s
- **列表滚动**: 60fps
- **安装包大小**: < 30MB

## 开发体验优化

### 开发工具链
- **代码检查**: ESLint + Prettier
- **类型检查**: TypeScript strict mode
- **Git Hooks**: Husky + lint-staged (提交前检查)
- **组件文档**: Storybook（可选，后期加入）
- **测试**: Vitest + React Testing Library

### AI 辅助开发优化
- 使用 TypeScript 让 AI 更准确生成代码
- 遵循 Ant Design 组件库，AI 可直接生成 UI 代码
- 明确的文件结构和命名规范，提升 AI 代码生成质量

## 相关决策
- ADR 0001: 采用微服务架构（前端调用 API Gateway）
- ADR 0006: API 设计规范 - RESTful + GraphQL（待定）

## 参考资料
- [React 官方文档](https://react.dev/)
- [Ant Design Pro](https://pro.ant.design/) - 企业级中台前端解决方案
- [React Query 文档](https://tanstack.com/query/latest)
- [React Native 性能优化](https://reactnative.dev/docs/performance)
