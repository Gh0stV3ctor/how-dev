# 生产场景模式——常见架构组合的真实技术栈

前面五篇讲清了架构的每一层和每一步。本文展示四个真实生产中常见的**技术栈组合模式**——不是说"你应该用这个"，而是说"你看，这些知名的项目用了这个组合，它们的选择是因为…"。

每个模式给：
- 技术栈列表（每层选了什么）
- 架构图（代码 → 部署的完整路径）
- 一个知名实例
- 适合的场景和不适合的场景

---

## 模式 1：传统单机部署——一个仓库、一台服务器

### 技术栈

| 层 | 选择 | 原因 |
|----|------|------|
| 前端 | React（Vite） | 社区最广、组件库最丰富 |
| 后端 | Go | 编译成单一二进制、部署简单、并发性能好 |
| 数据库 | PostgreSQL | 最通用、最可靠 |
| 缓存 | Redis | Session 存储 + 热点数据缓存 |
| 部署 | VPS + Docker Compose | 一台服务器搞定一切 |
| Nginx | 路由 + HTTPS + 静态文件 | 单一入口 |

### 架构图

```
用户浏览器
    │
    ▼ HTTPS (example.com)
┌─────────────────────────────────────┐
│ VPS (203.0.113.5)                    │
│                                      │
│ docker-compose.yml                  │
│                                      │
│ ┌──────────┐  ┌──────────┐         │
│ │ Nginx    │  │ Certbot   │         │
│ │ (80/443) │  │ (自动证书) │         │
│ └────┬─────┘  └──────────┘         │
│      │                               │
│      ├── / → React 静态文件           │
│      └── /api/* → Go:3001           │
│                   │                  │
│      ┌────────────┼──────────┐      │
│      ▼            ▼          ▼      │
│ ┌────────┐ ┌──────────┐ ┌──────┐   │
│ │  Go    │ │PostgreSQL│ │Redis │   │
│ │ :3001  │ │ :5432    │ │:6379 │   │
│ └────────┘ └──────────┘ └──────┘   │
└─────────────────────────────────────┘
```

### 实例：Focalboard（22k stars）

Focalboard 是 Mattermost 团队做的开源看板工具，技术栈：
- React 前端（TypeScript）
- Go 后端（gorilla/mux + 自定义 SQL builder）
- PostgreSQL / SQLite 可选
- 单二进制部署——下载一个文件，运行，就能用

**为什么选这个组合**：Focalboard 是为中小团队做的自部署工具。单二进制部署是竞争力——用户不需要会配置 Node.js、Python、数据库。下载、运行、浏览器打开就是全功能看板。

### 适合场景
- 团队给企业做自部署软件（用户不想配环境）
- 个人项目——一台 VPS 全搞定，月费 $5-20
- 对性能有要求但不至于需要 K8s 多机编排

### 不适合场景
- 弹性扩缩容——流量涨了只能升级服务器（纵向扩展），不可能"加两台机器"（横向扩展）
- 零停机部署——升级时会有短暂的连接中断

---

## 模式 2：全栈 SSR——Next.js + Vercel/Serverless

### 技术栈

| 层 | 选择 | 原因 |
|----|------|------|
| 渲染 | Next.js（React Server Components） | 前后端在同一个项目，用 RSC 在服务器上查询数据 |
| 后端 | Next.js API Routes | Serverless——每个 API 端点是一个独立的函数 |
| 类型共享 | 同项目 TypeScript | 前端和后端用同一个类型定义 |
| 数据库 | PostgreSQL（Neon / Supabase） | Serverless 友好的 PG 托管 |
| 认证 | NextAuth / Clerk | 与 Next.js 深度集成 |
| 部署 | Vercel | 推送即部署，自动 HTTPS，全球 CDN |

### 架构图

```
用户浏览器
    │
    ├── /（首页）
    │   → Vercel CDN → Serverless 函数（执行 RSC）
    │   → 服务器端查数据库
    │   → 返回渲染好的 HTML
    │   → 后续页面切换在客户端路由（不重新全量加载）
    │
    └── /api/*（API 请求）
        → Vercel CDN → Serverless 函数 → 查数据库 → 返回 JSON

数据库（Neon / Supabase）在 Vercel 外部——通过连接池访问
```

### 实例：ixartz SaaS Boilerplate（7.2k stars）

- Next.js + TypeScript + Drizzle ORM + Clerk + Tailwind + shadcn/ui
- 多租户 + RBAC + 团队管理
- 部署到 Vercel → 开发者只需 `git push`

**为什么选这个组合**：一个人或小团队做 SaaS。最大的价值是"零运维"和"快速迭代"。TypeScript 一次编译（lint/build/Vercel deploy）就上线。

### 适合场景
- 独立开发者的 SaaS——最高效的开发→上线路径
- 内容驱动的网站——SSR 让你 SEO 友好，同时保持交互
- 需要快速验证商业想法的 MVP

### 不适合场景
- 后端有长时间运行的任务（视频处理、ML 推理）——Serverless 有时间限制
- 极度依赖 WebSocket——Serverless 不适合长连接
- 需要完全控制服务器环境——Vercel 限制了你能做的配置

---

## 模式 3：静态站点生成（SSG）——不需要传统后端

### 技术栈

| 层 | 选择 | 原因 |
|----|------|------|
| 框架 | Astro / Next.js SSG / Hugo | 构建时生成纯 HTML——不需要服务器跑进程 |
| CSS | Tailwind / CSS Modules | — |
| 内容管理 | Strapi / Contentful / Markdown 文件 | 非开发者可以编辑内容 |
| 部署 | Cloudflare Pages / Vercel / Netlify | 纯静态——任何静态托管都能用 |

### 架构图

```
开发者 (内容编辑)
    │ Strapi 后台 → 修改内容 → 触发 Webhook
    ▼
GitHub Actions
    │ 检出 Repo → 拉取 CMS 内容 → 执行构建（npm run build）
    │ → 生成纯静态 HTML/CSS/JS → 部署到 CDN
    ▼
用户浏览器 → CDN → 纯静态 HTML

没有传统后端——每个用户看到的是预构建的 HTML。
评论/搜索等功能通过第三方服务嵌入（Disqus、Algolia）。
```

### 实例：文档站点（Vite、Tailwind、shadcn/ui 的官方文档都用这个模式）

大部分开源项目的文档站都用 Astro + Starlight 或 Nextra（Next.js 文档框架）构建——内容在 Markdown 里，构建时生成 HTML。

### 适合场景
- 文档站、博客、作品集、营销网站——内容变化慢，不需要频繁"动态计算"
- SEO 极为重要的网站——静态 HTML 是所有搜索引擎的最爱
- 流量很大但不需要个性化——CDN 直接返回 HTML，百万访问量无所谓

### 不适合场景
- 需要用户登录——用户看不到"自己的数据"，因为没有后端在运行
- 需要动态内容——你每发一篇新文章都要重新构建 + 部署

---

## 模式 4：应用 + 后台分离——前端 app + 后台管理系统

### 技术栈

| 层 | 选择 | 原因 |
|----|------|------|
| 前端（APP） | Next.js | 用户看到的网页/App |
| 前端（后台） | Next.js + Refine（管理框架） | 内部管理——用户管理、数据分析、内容审核 |
| 后端（API） | Go（面向用户的 API）+ Python（后台任务） | Go 处理并发 API，Python 处理数据管道 |
| 数据库 | PostgreSQL | — |
| 缓存/队列 | Redis + RabbitMQ | — |

### 架构图

```
用户 → Nginx → Go API (:3001) → PostgreSQL
                           │
               后台管理员 → Nginx → Next.js 后台 (:3002)
                               │ → Go API (相同)
                               │ → 后台专属数据查询（大 SQL）

                       RabbitMQ → Python Worker (后台任务)
```

### 实例：Plane（32k stars）

- 前端：React（用户看到的项目管理界面）+ React（管理后台）
- 后端：Python Django（API + 任务处理）
- 数据库：PostgreSQL
- 缓存/队列：Redis + Celery
- 部署：Docker Compose（单机）或 K8s（集群）

**为什么选这个组合**：项目管理工具的用户界面很重（拖拽看板、甘特图），管理后台的数据分析需求不同（统计报表）。两个前端共享同一个 Django API，但各自优化各自的体验。

### 适合场景
- 用户 App 和内部后台的界面需求差异很大
- 后端有多种工作负载：快速 API（Go）+ 重计算（Python Worker）
- 团队超过 5 人，前端和后端有明确的分工

### 不适合场景
- 小团队 → 维护多套前端 + 多语言后端 → 性价比低

---

## 四种模式的选型对比

| | 单机部署 | Next.js SSR | SSG | 前后台分离 |
|---|---|---|---|---|
| **团队** | 1-3 人 | 1-5 人 | 1-2 人 | 5+ 人 |
| **运维负担** | 低（Docker Compose） | 极低（Vercel） | 极低 | 中到高 |
| **前端** | React SPA | Next.js 全栈（RSC） | Astro/Hugo SSG | Next.js APP + 后台 |
| **后端** | Go（单进程） | Serverless Functions | 无（CMS提供） | 多语言/多服务 |
| **数据库** | PG + Redis | 托管 PG | 无/第三方 | PG 独立 |
| **开发速度** | ★★★★ | ★★★★★ | ★★★★★ | ★★★ |
| **最大流量** | 单机瓶颈（~1-5k QPS） | 平台自动扩缩容 | CDN 无限 | 横向扩缩容 |
| **月费** | ~$10-50（VPS） | ~$20-200（Vercel/托管DB） | ~$0-20（纯托管） | ~$100-2000+（多服务） |

---

## 你该选哪个

```
从你的约束开始：
  "一个人、做第一个 SaaS、想快速看到结果、月预算 <$50"
    → 模式 2（Next.js + Vercel）——一个 git push 就上线

  "一个人、用 Go/Rust 写后端、想要完全控制服务器"
    → 模式 1（单机 Docker Compose）——一个 docker-compose up 全搞定

  "团队、有专属运维、需要支持大量用户"
    → 模式 4（前后台分离）——分层后各自独立维护和扩缩容

  "一个简单的文档或博客——只是展示内容，不需要应用逻辑"
    → 模式 3（SSG）——Markdown 文件文章、Git push 自动发布、零成本
```

---

## 下一章

这些模式中提到的"隐形"基础设施——npm、Docker Hub、Vercel、GitHub Actions——你自己不需要搭建，但它们是怎么运作的？

→ [07 生态基础设施](07-ecosystem-infra.md)
