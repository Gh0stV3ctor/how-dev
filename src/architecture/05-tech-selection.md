# 技术选型——在每个生态位上怎么选择

## 技术选型不是"选最好的"

没有"最好的语言"、"最好的框架"、"最好的数据库"。**每一个选择都是一个 trade-off——你得到一些东西，同时付出另一些代价。**

技术选型的核心问题是：**在你的具体约束下，哪个 trade-off 组合最划算？**

约束包括：
- 团队会什么（一个 C++ 团队不会因为"Go 更适合 Web"突然变成 Go 团队）
- 做什么（实时协作 App 和博客站点的性能需求完全不同）
- 什么时候交付（三个月上线和下周上线，技术选择不同）
- 多少预算（自建 K8s 集群 vs 用 Vercel 托管）

---

## 选型坐标系：性能 vs 开发速度

所有技术选型都在这个二维空间中有自己的位置：

```
开发速度快 ↑
             │   Python（Django/FastAPI）
             │   JavaScript/TypeScript（Node.js）
             │   Ruby（Rails）
             │   PHP（Laravel）
             │
             │   Go
             │   Java / C#（Spring Boot / ASP.NET）
             │
             │   Rust
             │   C / C++
             │
             └────────────────────────→ 运行时性能高

纵轴：写同样功能需要的时间。横轴：代码在服务器上的运行效率。
```

**但这张图是简化的。** 实际情况更复杂：

- .Python 写起来快，但服务 1 万并发时需要 20 台服务器。Go 写起来慢一些，但 2 台就够了。长期来看，Go 的硬件成本可能更低——但这个"长期"可能永远不出现
- Rust 的性能最高，但招聘 Rust 开发者比招聘 Node.js 开发者难 10 倍。如果你的业务瓶颈是市场推广而不是服务器性能，花精力招 Rust 开发者的价值在哪里？

---

## 后端语言——逐门剖析

### Go

**适合**：API 服务、微服务、CLI 工具、网络中间件、DevOps 工具

**为什么选它**：
- 一个命令编译成一个文件——部署时复制这一个文件就行
- Goroutine——轻松处理数万并发连接，而且语法简单（`go func()` 就启动了一个协程）
- 编译快——大型项目几秒编译完成，反馈快
- 标准库质量高——`net/http` 可以直接用在生产环境，很多项目用标准库就够了
- 静态类型 + 隐式接口——类型安全但不繁琐

**为什么选别的**：
- 生态不够成熟——Go 的 Web 框架没有一个像 Django/Rails 那样"30 分钟出一个全栈"
- 没有泛型支持（Go 1.18 才引入泛型），一些模式（如通用 ORM）实现起来麻烦
- 错误处理冗长——`if err != nil` 到处都是，没有 Rust 的 `?` 省事

**代表作**：Docker、Kubernetes、Caddy、PocketBase、Terraform

### Node.js + TypeScript

**适合**：前后端同一个人、需要快速迭代的 SaaS、实时 WebSocket 应用、API 网关

**为什么选它**：
- 同一种语言——写前端的是 TypeScript，写后端的也是 TypeScript
- NPM 生态是最大的——几乎所有功能都有现成包
- 非阻塞 I/O——处理大量并发 I/O（数据库查询、外部 API 调用）时不需要多线程
- TypeScript 的类型系统提升了对大型项目的支撑

**为什么选别的**：
- 单线程模型——CPU 密集型任务（图片处理、加密）会阻塞整个进程
- `node_modules/` 的体积和依赖地狱
- 运行时性能比 Go/Rust/C++ 差——对性能极端敏感的服务不合适
- TypeScript 配置和工具链比 Go/Rust 复杂

**代表作**：VS Code Server、Next.js、Payload CMS、tRPC、Prisma

### Python

**适合**：数据科学、AI/ML 后端、快速原型、自动化脚本、以 Django 为核心的传统 CRUD 后端

**为什么选它**：
- 学习曲线最低——语法简洁，新手最快的上手速度
- AI/ML 的统治地位——如果你是 AI 产品，Python 后端是最自然的选择
- Django 内置一切——ORM、Admin 面板、认证、表单——"开箱即用"的最强实现
- 可读性——Python 代码就是文档

**为什么选别的**：
- 性能——CPython 的单线程吞吐量比 Go/Rust 低一个数量级
- GIL（全局解释器锁）——多线程受限，高并发需要多进程（内存开销大）
- 类型检查是后加的（mypy/Pydantic）——不是语言原生
- 依赖管理生态骚乱——pip→Pipenv→Poetry→uv→rye——历史包袱大

**代表作**：Instagram（Django）、Sentry、PostHog、Plane、yt-dlp

### Rust

**适合**：性能关键的服务（API 网关、代理、数据库内部、编译工具）、嵌入式系统、CLI 工具

**为什么选它**：
- 性能 = C/C++ 级别，安全 = 编译期保证（没有 segfault、没有 data race）
- 零运行时开销——适合做其他语言的底层基础设施
- 类型系统无妥协——所有权（Ownership）+ 借用（Borrowing）+ 生命周期（Lifetime）在编译期消除整类 bug
- Cargo 是优秀的构建+包管理器——一个工具干所有事

**为什么选别的**：
- 学习曲线极陡——所有权和借用不直观，编译器和开发者较劲
- 编译慢——大型项目可能编译几十秒甚至几分钟（增量编译缓解但不能消除）
- 生态仍在成熟——Web 框架有几家竞争（Axum / Actix-web / Rocket），没有明确的主流
- 招聘 Rust 开发者 = 比 Go/Node.js 难一个数量级

**代表作**：ripgrep、fd、bat、Meilisearch、Lemmy、Zed、Deno（部分用 Rust）

### Java / C# (.NET)

**适合**：企业级应用（银行、保险、ERP）、大型后端系统、Android App 后端

**为什么选它**：
- 生态成熟——Spring Boot（Java）和 ASP.NET Core（C#）是经历了 20 年打磨的生产框架
- JVM/CLR 的 JIT 编译——长期稳定运行的性能不输 Go
- 强类型 + 庞大的 IDE 支持——企业级开发的生产力工具极成熟
- 招聘容易——Java 开发者数量全球最高

**为什么选别的**：
- 启动慢、内存占用高——不适合 Serverless（冷启动可能数秒）、资源受限环境
- 笨重——"Hello World"需要 project setup、依赖解析、配置一堆
- 云原生时代的心智模型不匹配——12-factor app、容器化、不可变基础设施这些概念是在 JVM 时代之后确立的
- Kotlin/Scala（JVM 上的"更好语言"）降低了 Java 本身的吸引力

**代表作**：Elasticsearch（Java）、Apache Kafka（Java）、Keycloak（Java）、Signal Server（Java）

---

## 前端框架——选择的不是语法，是响应式模型

React 和 Vue 和 Svelte 的差别不是一个"用 JSX"一个"用 template"——是**底层响应式模型的不同**。

| | React | Vue 3 | Svelte |
|---|---|---|---|
| **响应式模型** | Virtual DOM + Diff | 细粒度响应式（Proxy） | 编译器优化（构建时分析） |
| **状态变化时** | 重新执行整个组件函数 → 生成新 VDOM → diff → 更新 DOM | 精确追踪"这个状态变量被哪些 DOM 节点依赖" → 只更新那些节点 | 构建时生成直接操作 DOM 的代码 → 运行时就是普通 JS |
| **代码组织** | JSX（JS 里写 HTML） | SFC（HTML + CSS + JS 在一个文件里） | SFC（类似 Vue） |
| **学习曲线** | 中等（JSX + hooks + 大量第三方方案） | 中低（模板语法直观，方案更内聚） | 低到中等（接近 HTML/CSS/JS） |
| **生态成熟度** | ★★★★★ | ★★★★ | ★★★ |

**如何选择**：第一次做前端 → React（生态最丰富，出问题能搜到答案）。追求极致性能和简单 → Svelte。想要 React 的生态 + Vue 的简洁 → 看 SolidJS。

---

## 数据库——各司其职，不是取代

| 数据库 | 类型 | 适合场景 | 不适合场景 |
|--------|------|---------|-----------|
| **PostgreSQL** | 关系型 | 几乎一切——ACID 事务、复杂查询、文档存储（JSONB）、全文搜索、地理空间 | 海量时序数据（专门引擎更快） |
| **SQLite** | 嵌入式关系型 | 单机应用、移动 App、边缘设备、开发阶段 | 多进程并发写入——SQLite 只适合低并发写入 |
| **MySQL/MariaDB** | 关系型 | 和 PostgreSQL 相似，在读密集场景略优 | 功能不如 PG 丰富（CTE、窗口函数晚很多） |
| **MongoDB** | 文档型（NoSQL） | 快速原型（schema 自由）、嵌套文档 | 复杂 JOIN、ACID 事务需求 |
| **Redis** | 内存 KV + 数据结构 | 缓存、Session 存储、消息队列（Pub/Sub）、限流、实时排行榜 | 持久化数据——Redis 是辅助，不是主存储 |
| **Elasticsearch** | 全文搜索引擎 | 产品搜索、日志分析、聚合查询 | 不用作主要数据库（延迟、一致性） |

**关键认识**：PostgreSQL 是新项目的默认选择。不要用 MongoDB 只是因为"不用写 SQL"——SQL 的价值远大于它初始的学习成本。

---

## 部署平台——选择的是"你管多少、平台管多少"

见 [04 部署全景](04-deployment.md) 结尾的抽象层级图。

| 你做什么 | 平台 | 适合 |
|---------|------|------|
| 只管代码，不管任何运维 | Vercel / Netlify | 前端 + Serverless 后端、小团队、快速迭代 |
| 管环境变量和构建，不管 OS | Fly.io / Railway / Render | 全栈应用、Node.js / Go，不想运维 |
| 管 Docker，不管 K8s | 自己 VPS + Docker Compose | 需要完全控制但不想搞集群 |
| 管一切 | 自己 VPS + systemd | 需要每一点性能、完全控制 |
| 管 K8s 集群 | AWS EKS / GKE | 大型团队、微服务、弹性扩缩容 |

---

## 选型决策流程

不是"先学完所有选项再选"——是从你的约束出发，逐步缩小范围：

```
1. 你要做什么类型的应用？
   ├── 纯内容（博客、文档站）→ 静态生成（Astro、Next.js SSG）→ 不需要传统后端
   ├── SaaS（用户登录、操作数据）→ 全栈应用 → 需要后端 + 数据库
   └── API 服务（给别人用）→ 纯后端 → 不需要前端

2. 你有多长时间？
   ├── 1-2 周 → 快速原型 → Python FastAPI / Node.js + Next.js
   └── 1+ 月 → 有打磨时间 → Go / Rust 更稳定的类型安全基础

3. 团队会什么？
   ├── 只会 JS/TS → Node.js 后端是最自然的
   └── 会多种语言 → 根据场景选（API = Go，AI = Python，实时 = Node.js）

4. 预算和流量？
   ├── 小流量、创业 → Vercel + PostgreSQL（最简单）
   └── 大流量、成本敏感 → 自己 VPS + Docker（服务器成本低）

5. 验证选择的连贯性
   ├── 前后端技术栈的部署要协调——Next.js + Go 需要两个平台
   └── 数据库和 ORM 要匹配——Prisma + PostgreSQL，SQLx + Rust
```

---

## 下一章

理论讲完了。来看真实项目中的技术栈组合——同一类需求，不同的团队做出了不同的选择。

→ [06 生产场景模式](06-production-patterns.md)
