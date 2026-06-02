# 全栈项目参考资源对比

> 本文调研了当前（2024-2026）最值得作为代码学习参考的开源全栈项目。筛选标准：架构质量高、技术栈现代、活跃维护、可作为"参考答案"阅读。

---

## 一、Next.js / TypeScript 生态（最匹配推荐技术栈）

### 旗舰参考项目

| 项目 | Stars | 是什么 | 技术栈 |
|------|-------|--------|--------|
| **cal.diy** | 42.4k | 开源版 Calendly——日程预约 | Next.js + TS + Prisma + PostgreSQL + tRPC + Tailwind + NextAuth + Turborepo |
| **Supabase** | 102k | 开源版 Firebase——数据库+认证+实时+存储 | Next.js（管理后台）+ TS + PostgreSQL + Deno + Go |
| **Payload CMS** | 42.3k | 基于 Next.js 的 CMS 框架 | Next.js + TS + PostgreSQL/MongoDB + GraphQL + REST |
| **Dify** | 141k | AI 工作流平台——RAG、Agent 编排 | Next.js（前端）+ TS + Python（后端） |
| **tRPC** | 40.2k | 端到端类型安全的 API 框架 | TS + React + Next.js 适配 |
| **shadcn/ui** | 114k | 组件代码分发平台 | TS + React + Tailwind + Radix |
| **Refine** | 34.7k | React 框架——管理后台、仪表盘、B2B 应用 | TS + React + 多 UI 适配器（AntD/MUI/Headless） |
| **Homepage** | 30k | 个人/团队仪表盘主页 | JS + React + Docker + Node.js |
| **Postiz** | 30.3k | 社交媒体定时发布工具 | TS + Next.js + Redis |

### SaaS 脚手架/Starter

| 项目 | Stars | 技术栈 | 特色 |
|------|-------|--------|------|
| **ixartz/SaaS-Boilerplate** | 7.1k | Next.js + TS + Tailwind + shadcn/ui | 多租户、认证、订阅——社区最认可的 Next.js SaaS 起点 |
| **T3 Stack (create-t3-app)** | 26k+ | Next.js + TS + tRPC + Prisma + Tailwind + NextAuth | 不是项目，是脚手架——但它定义了现代 TS 全栈的标准架构模式 |

### 学习价值评估

| 项目 | 适合初学者读吗 | 能学到什么 | 坑 |
|------|-------------|-----------|-----|
| **cal.diy** | 🟡 中（代码量大但结构清晰） | T3 Stack 生产实践、Monorepo 组织、Prisma 高级用法、tRPC 架构、日历调度复杂业务逻辑 | 项目太大，Monorepo 初学者可能迷失 |
| **Supabase** | 🔴 难（多服务架构） | 实时订阅、RLS、PostgREST 思想、Edge Functions、大型开源项目的组织 | 太复杂，不是"项目"而是"平台" |
| **Payload CMS** | 🟡 中 | CMS 架构、插件系统、自动生成 API、访问控制 | 框架级别，不是应用级别 |
| **Dify** | 🔴 难 | AI 编排、RAG、多服务通信 | Python + TS 混合，对初学者太重 |
| **ixartz 脚手架** | 🟢 易 | 开箱即用的 SaaS 骨架——认证、多租户、订阅、数据库的完整模式 | 可能太"模板化"——学不到架构决策的 WHY |
| **Homepage** | 🟢 易（3 万星但项目小） | Widget 架构、Docker API 集成、服务发现、YAML 驱动配置 | 侧重配置驱动，不侧重用户交互 |

### 该项目类型的结论

**cal.diy 是唯一一个同时满足"架构质量高、技术栈完全匹配、有复杂业务逻辑、足够知名"的 Next.js 全栈参考项目。** 但它 42.4k stars 代码量庞大——适合作为"遇到问题时翻阅的参考答案"，不适合从头到尾读完。

对于初学者，**学习路径应该是**：
1. 用 `create-t3-app` 搭建项目骨架（它内置了 cal.diy 同款架构的精简版）
2. 做自己的项目
3. 遇到"数据库关系怎么设计"、"认证流程怎么写"时，翻阅 cal.diy 的对应代码

---

## 二、Go 生态

Go 的全栈项目风格和 JS 生态截然不同——更倾向于"小而精"、无框架、标准库优先。

| 项目 | Stars | 是什么 | 技术栈 | 架构学习价值 |
|------|-------|--------|--------|------------|
| **PocketBase** | 42k+ | 一键部署的后端——SQLite + 文件存储 + 认证 + 管理后台 | Go + SQLite + Svelte（管理后台） | ★★★★★ 单个 Go 二进制文件包含完整后端——轻量全栈的极致示范 |
| **Gitea** | 47k+ | 自部署 Git 服务（GitHub 替代） | Go + 模板渲染 + PostgreSQL/MySQL/SQLite | ★★★★ Go Web 项目组织 + Git 底层调用 + 大规模测试 |
| **Mattermost** | 30k+ | 开源版 Slack | Go（后端）+ React（前端）+ PostgreSQL | ★★★★ 大型 Go + React 全栈项目的协作模式 |
| **Caddy** | 60k+ | 自动 HTTPS 的 Web 服务器 | Go | ★★★ 中间件模式、ACME 协议、Go 的扩展架构 |
| **Focalboard** | 22k+ | 开源版 Notion/看板 | Go（后端）+ React（前端）+ PostgreSQL/SQLite | ★★★★★ 如果你做任务管理，这是最直接的 Go 语言参考 |

**Go 生态特点**：没有"元框架"概念。Go 社区不相信大一统框架——他们用标准库 `net/http` + 几个小库组合。代码组织方式比 JS 生态统一得多，但也更依赖开发者的架构判断力。

---

## 三、Rust 生态

Rust 全栈项目偏少，但质量通常很高。

| 项目 | Stars | 是什么 | 技术栈 | 架构学习价值 |
|------|-------|--------|--------|------------|
| **Lemmy** | 13k+ | 开源版 Reddit——联邦式链接聚合 | Rust（后端 Actix+Diesel）+ TypeScript（前端）+ PostgreSQL | ★★★★★ 完整的社区平台——投票、评论、联邦协议 |
| **Meilisearch** | 48k+ | 全文搜索引擎 | Rust + 自建 HTTP 层 | ★★★★ 搜索引擎内部架构、异步 I/O、索引结构 |
| **Zed** | 52k+ | GPU 加速的协作代码编辑器 | Rust + GPUI（自研 GUI 框架） | ★★★ 系统级 GUI 架构，偏底层 |
| **SurrealDB** | 28k+ | 多模型数据库 | Rust | ★★★ 数据库内部架构，偏底层 |
| **poem-admin** | 12 stars | Rust 管理后台参考 | Poem + SQLx + Casbin RBAC | ★★★★ 小但完整——适合初学者阅读 |

**Rust 生态特点**：Rust 的 Web 生态仍在成熟中。Axum（Tokio 团队出品）是当前最受推荐的 Web 框架。适合"已经有其他语言全栈经验后学 Rust"的场景，不适合作为第一个全栈项目。

---

## 四、Python 生态

| 项目 | Stars | 是什么 | 技术栈 | 架构学习价值 |
|------|-------|--------|--------|------------|
| **Sentry** | 40k+ | 错误追踪平台 | Python/Django（后端）+ React（前端）+ PostgreSQL + Redis + Kafka | ★★★★★ 大型 Python 生产项目的工业标准 |
| **PostHog** | 22k+ | 开源产品分析平台 | Python/Django（后端）+ React（前端）+ ClickHouse + Kafka | ★★★★★ 事件驱动的分析管线架构 |
| **Plane** | 32k+ | 开源版 Jira——项目管理 | Python/Django（后端）+ React（前端）+ PostgreSQL + Redis | ★★★★★ 如果你做任务管理，这是最直接的全栈参考 |
| **Taiga** | 13k+ | 敏捷项目管理 | Python/Django（后端）+ Angular（前端）+ PostgreSQL | ★★★ 老牌项目管理工具，代码风格偏老 |

**Python 生态特点**：Django 仍是 Python 后端的事实标准。FastAPI 增长极快但大型 OSS 项目仍以 Django 为主。Python 后端通常配 React 前端——这是最常见的"跨语言全栈"组合。

---

## 五、按你的选题方向推荐参考项目

### 做 Code Snippet Manager

| 参考项目 | 学它的什么 |
|---------|-----------|
| **cal.diy** | 项目结构、Prisma Schema 设计、NextAuth 认证、tRPC 类型安全 API——最核心的架构参考 |
| **shadcn/ui** | 代码分发模式——shadcn 怎么让用户"复制代码到项目"而不是"安装 npm 包"。片段管理器的"分享片段"可以借鉴 |
| **ixartz SaaS Boilerplate** | 认证+数据库+多租户的骨架模式 |

### 做任务管理

| 参考项目 | 学它的什么 |
|---------|-----------|
| **Plane**（32k stars, Python/Django + React） | 任务管理的完整数据模型——项目/任务/标签/评论/权限的 schema 设计 |
| **Focalboard**（22k stars, Go + React） | 看板视图的拖拽逻辑、实时同步 |
| **cal.diy** | 全栈架构模式 |

### 做个人仪表盘

| 参考项目 | 学它的什么 |
|---------|-----------|
| **Homepage**（30k stars, JS + React） | Widget 插件架构、Docker API 集成、服务发现 |
| **cal.diy** | 全栈架构模式 |

### 做团队公告板

| 参考项目 | 学它的什么 |
|---------|-----------|
| **Mattermost**（30k stars） | 消息通知系统架构、已读/未读追踪 |
| **Plane** | 团队协作的权限模型 |

---

## 六、额外的学习资源聚合

### 项目结构参考（不写代码，只看组织方式）

| 资源 | 是什么 |
|------|--------|
| **[Bulletproof React](https://github.com/alan2207/bulletproof-react)** | React 项目的文件结构和架构模式指南——27k stars |
| **[T3 Stack 文档](https://create.t3.gg/)** | Next.js + TS + tRPC + Prisma 的标准项目骨架 |
| **[Next.js 官方示例](https://github.com/vercel/next.js/tree/canary/examples)** | Vercel 官方的各种场景示例（含 `with-prisma`、`next-auth` 等） |

### 全栈挑战/教程类（有参考答案的练习题）

| 资源 | 是什么 |
|------|--------|
| **[Fullstack Open](https://fullstackopen.com/)** | 赫尔辛基大学的免费全栈课程——React + Node.js，有完整项目练习 |
| **[RealWorld App](https://github.com/gothinkster/realworld)** | "Medium 克隆"——同一个项目用 20+ 种技术栈实现。你可以对比同一功能在 Next.js/Go/Rust/Django 中的不同写法 |

**RealWorld 特别值得注意**：它不是"最佳实践"而是"中等复杂度真实应用的常见写法"。每个技术栈的实现在 1 万行代码左右，正好适合阅读。而且你可以对比同一个功能（比如"用户关注"）在不同技术栈中怎么实现。

---

## 七、如果你要选一个"唯一参考"

按你的约束（第一个项目、Next.js + TS、边做边学、需要参考答案）：

```
第一参考：cal.diy（42.4k stars）
  → 遇到具体问题时翻阅对应文件：
    - "Auth 怎么写的" → packages/features/auth/
    - "数据库关系怎么设计" → packages/prisma/schema.prisma
    - "API 层怎么组织" → packages/trpc/
    - "组件怎么拆分" → packages/features/

第二参考：Bulletproof React（27k stars）
  → 项目开始前先通读一遍，理解"好的 React 项目长什么样"

脚手架：create-t3-app
  → 不要从零搭项目，用它初始化。它给你的结构和 cal.diy 是同源的

跨技术栈对比：RealWorld App
  → 想看"同一个功能在 Go/Rust/Python 中怎么写"时查阅
```

---

## 八、一句话总结

**做 Code Snippet Manager + 用 create-t3-app 初始化 + 读 cal.diy 作为架构参考答案。** 不是因为这三个是完美的，而是因为它们在"学习价值、代码质量、生态匹配度、初学者友好度"四个维度上综合得分最高。
