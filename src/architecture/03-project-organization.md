# 项目组织——前后端代码的物理关系

## 核心问题

你打开编辑器，要写一个 Web 应用。前端代码（React 组件、CSS）和后端代码（API 路由、数据库查询）——它们该放在几个文件夹里？

答案不是"放一个文件夹更方便"或"分开放更专业"。答案取决于：**团队规模、技术栈选择、部署目标**。

---

## 三种组织结构

### 模式 A：分仓库——前后端完全独立

```
github.com/yourname/my-app-frontend/        ← Git 仓库 1
├── src/
│   ├── app/
│   ├── components/
│   └── lib/
├── package.json           ← React + Vite + Tailwind
├── tsconfig.json
└── vite.config.ts

github.com/yourname/my-app-backend/         ← Git 仓库 2
├── cmd/
│   └── server/main.go
├── internal/
├── go.mod                  ← Go 项目
└── Dockerfile
```

**工作流**：

```
前端开发者：cd my-app-frontend → npm run dev → localhost:5173
后端开发者：cd my-app-backend → go run . → localhost:8080

前端代码里写的是：
  const API_URL = "http://localhost:8080/api";  // 开发时
  // const API_URL = "https://api.example.com";  // 生产时
```

**部署**：

```
前端：
  npm run build → dist/ → 上传到 CDN 或 Vercel
后端：
  go build → server → Docker 镜像 → 部署到 AWS/k8s
两者部署完全独立——前端可以一天部署十次，后端一个月一次
```

**优点**：
- 前后端可以独立部署、独立扩缩容
- 前后端团队可以用不同的 CI/CD 流水线
- 前端用 Vercel，后端用 AWS——各自选最适合的平台
- 不会出现"改了前端 CSS 触发后端重新构建"或反过来

**缺点**：
- API 变更时，前端和后端要同步——"后端改了字段名，前端没人通知，突然页面炸了"
- 没有共享类型定义——前端定义 `User` 接口、后端也定义同一个 `User` 结构体，可能不一致
- 需要两个独立的 CI/CD 流水线维护

**适合**：团队规模 >3 人，前后端技术栈不同（Go 后端 + React 前端），独立部署需求明确。

---

### 模式 B：Monorepo——同仓库、分离构建

```
my-project/                              ← 一个 Git 仓库
├── apps/
│   ├── frontend/                        ← Next.js 项目
│   │   ├── src/
│   │   ├── package.json
│   │   └── tsconfig.json
│   └── backend/                         ← Go 项目
│       ├── cmd/server/
│       ├── go.mod
│       └── Dockerfile
├── packages/
│   └── shared-types/                    ← 共享的类型定义
│       ├── src/
│       │   └── user.ts                  ← User 接口（前后端都用）
│       └── package.json
├── package.json                         ← 工作空间根配置
└── turbo.json                           ← Turborepo 构建编排
```

**Turborepo（构建编排器）的作用**：

```
你运行 turbo build：
  1. 检查 packages/shared-types 有没有改 → 有 → 先构建 shared-types
  2. 检查 apps/frontend 有没有改 → 有 → 构建 frontend
  3. 检查 apps/backend 有没有改 → 有 → 构建 backend
  4. 如果都没改 → 用缓存，零秒跳过

你改了一行共享类型：
  1. shared-types 重新构建
  2. frontend 重新构建（因为它依赖 shared-types）
  3. backend 不重新构建（Go 不依赖 shared-types npm 包）
```

**部署**：

```
CI/CD 检测变更路径：
  只有 apps/frontend 变了 → 只部署前端
  只有 apps/backend 变了 → 只部署后端
  packages/shared-types 变了 → 两个都部署（因为两个都依赖它）
```

**优点**：
- API 类型共享：`packages/shared-types` 里的 `User` 接口，前后端都引用——改了它，两边都报错（编译时发现问题）
- 一个 PR 同时改前后端 + 类型定义——不会出现"前端发了 PR、后端还没改"的同步问题
- 统一的 CI/CD 流水线——但可以按路径差异化执行

**缺点**：
- 仓库很大（Node.js 项目有 `node_modules/`，Go 项目也有 `vendor/` 或下载缓存）
- 需要构建编排工具（Turborepo / Nx / Bazel）
- 初期的构建配置比"两个独立仓库"复杂

**适合**：团队 1-5 人，前后端同一个人（或紧密协作的小团队），需要快速迭代全栈。

**真实项目**：cal.diy（42k stars）就是 Turborepo Monorepo。

---

### 模式 C：全栈合一——Next.js 既是前端也是后端

```
my-app/                                ← 一个 Git 仓库 = 一个 Next.js 项目
├── src/
│   ├── app/
│   │   ├── page.tsx                   ← 前端：首页路由
│   │   ├── layout.tsx                 ← 前端：全局布局
│   │   ├── dashboard/
│   │   │   └── page.tsx               ← 前端：仪表盘路由
│   │   └── api/
│   │       ├── users/
│   │       │   └── route.ts           ← 后端：GET /api/users
│   │       └── auth/
│   │           └── route.ts           ← 后端：POST /api/auth/login
│   ├── components/                    ← 前端组件
│   │   └── Button.tsx
│   └── lib/
│       ├── db.ts                      ← 数据库连接（前端和后端都可能用到）
│       └── auth.ts                    ← 认证逻辑（后端用）
├── package.json                       ← 一个——前后端共享
└── next.config.ts
```

**"既是前端也是后端"的具体含义**：

```typescript
// src/app/page.tsx —— 这段代码是前端吗？是后端吗？
export default async function HomePage() {
  // 这段代码在服务器上执行（后端）——可以直接访问数据库
  const posts = await db.post.findMany({ take: 5 });

  // return 出去的东西会被序列化，发送给浏览器（前端）
  return (
    <div>
      {posts.map(post => (
        <article key={post.id}>{post.title}</article>
      ))}
    </div>
  );
}
```

一个文件里同时有"后端逻辑"（`await db.post.findMany()`）和"前端渲染"（`<div>` + Map），这种写法叫 React Server Component (RSC)。运行模式如下:

- 同一个组件文件中，**可以在服务器端执行数据库查询**
- 查询结果直接渲染为 HTML，**不需要 API fetch 往返**
- 但**不能用** `useState`、`useEffect`、`onClick`（这些是浏览器端的东西）
- 需要交互的组件加 `"use client"`——它就变成纯前端了

**部署**：Vercel `git push` → 自动检测 Next.js → 构建 + 部署

**优点**：
- 极简——一个项目、一个 `npm run dev` 就前后端都跑
- 类型共享天然——都在一个 TypeScript 项目里
- 部署最简单——Vercel 一键部署
- RSC 可以减少客户端 JS 体积——服务器渲染后发给浏览器的是 HTML，不是 JS 代码

**缺点**：
- 前后端紧耦合——不能"前端用 Vercel、后端用 AWS"
- 不适合大型团队——几十个人同时改同一个 `src/app/` 目录
- 后端能力受限于 Serverless（Vercel 上每个请求有限的时间和内存）

**真实项目**：许多现代 SaaS starter（ixartz/SaaS-Boilerplate）用这个模式。

---

## 三种模式的对比

| | 分仓库 | Monorepo | Next.js 全栈合一 |
|---|---|---|---|
| **仓库数量** | 2+ | 1 | 1 |
| **构建工具** | 各自独立 | Turborepo / Nx 编排 | Next.js 内置 |
| **类型共享** | 无（需额外维护 OpenAPI） | `packages/shared-types/` | 天然（同项目） |
| **部署独立性** | 完全独立 | 按路径触发 | 不独立（一起部署） |
| **适合团队规模** | 5+ | 2-10 | 1-3 |
| **后端语言可选** | 任何 | 任何 | 只能 Node.js（Serverless） |
| **学习成本** | 低（各管各） | 中（编排工具） | 低（单项目） |
| **生产案例** | 传统企业 | cal.diy, Vercel 自身 | 大量 indie SaaS |

---

## 如何选择

```
你是独自开发一个 SaaS？
  → 用 Next.js 全栈合一（模式 C）。部署最省心，开发最快。

你是小团队（2-5 人），前后端你一个人做但以后可能分给别人？
  → 用 Monorepo（模式 B）。今天你一个人改全栈，一个 PR 搞定。明天有人接手后端，路径自然分开了。

你已经有团队，前后端由不同的人负责？
  → 用分仓库（模式 A）。API 契约（OpenAPI）作为两者之间的正式协议。

你是大公司？
  → 你可能需要微服务 + 多个团队。本文档不是写给你的场景的。
```

---

## 下一章

代码组织好了，构建产物也出来了。下一步：**怎么让全世界访问到你的应用？**

→ [04 部署全景](04-deployment.md)
