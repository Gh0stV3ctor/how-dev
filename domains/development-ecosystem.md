# 开发生态大地图

## 为什么你感到混乱

当你说出"React、Vue、Angular、JS/TS、Java、Python、Django、Flutter、React Native、Kubernetes、Redis、Next.js、Kotlin、CSS、HTML、ASP.NET"这一串名字时，你的大脑同时调动了至少五种不同类别的概念。把它们平铺在一起当然混乱——就像把"发动机、SUV、柴油、轮胎、德国、驾校"当同类事物讨论。

本文的目标：给开发世界画一张完整的地图，把每个概念放到它所属的类别和位置上。看完之后，你能精准回答"X 是什么、为什么不是 Y、它和 Z 是什么关系"。

---

## 第一步：两个维度——平台和角色

所有开发领域都是两个维度的交叉：

- **平台（在哪跑）**：你的代码最终运行在什么设备上？
- **角色（干什么）**：你的代码负责呈现界面，还是处理数据逻辑？

```
                          角色
                  前端 (Frontend)          后端 (Backend)
               ┌─────────────────────┬─────────────────────┐
       Web     │ JS/TS + React/Vue/  │ Python/Django,      │
               │ Angular + HTML/CSS  │ Java/Spring,        │
               │                     │ C#/ASP.NET,         │
               │                     │ Go, Rust, PHP/Laravel│
               ├─────────────────────┼─────────────────────┤
平台   移动    │ Swift (iOS),        │                     │
               │ Kotlin (Android),   │   和 Web 后端       │
               │ Flutter (Dart),     │   完全一样          │
               │ React Native (JS)   │                     │
               ├─────────────────────┼─────────────────────┤
      桌面     │ Electron (JS),      │                     │
               │ Tauri (Rust),       │   和 Web 后端       │
               │ Qt (C++),           │   完全一样          │
               │ SwiftUI (macOS)     │                     │
               └─────────────────────┴─────────────────────┘
                          ↕ 两者之间通过网络通信
                          ↕ 两者之下有基础设施支撑

               ┌─────────────────────────────────────────┐
               │  基础设施（所有角色、所有平台共用）         │
               │  Docker, Kubernetes, Nginx,              │
               │  PostgreSQL, Redis, RabbitMQ, Kafka      │
               └─────────────────────────────────────────┘
```

**关键理解**：

- Web 前端、移动端、桌面端是**平行关系**——它们都是不同平台上的"前端"
- Web 后端是**所有平台前端共享的**——移动 App 和桌面 App 调用的 API 和 Web 前端调用的 API 可以是同一个后端
- 基础设施在**所有东西下面**——不管你用 Python 还是 Go，数据库都是 PostgreSQL；不管你是 Web 还是移动，容器都是 Docker

---

## 第二步：概念分类——语言、引擎、框架、元框架、组件库、SDK、基础设施

在把具体技术名字放上地图之前，必须先定义**概念类别**。否则你会以为"Flutter 和 React 是一类东西，因为都是前端"，但实际上 Flutter 是一个跨平台框架而 React 是 Web UI 框架——它们的类别不完全相同。

### 类别速查表

| 类别 | 定义 | 例子 | 判断标准 |
|------|------|------|---------|
| **编程语言** | 你写代码用的语法和语义规则 | JavaScript, Python, Java, Kotlin, Dart, C#, Swift, Go, Rust, C, C++, PHP, Ruby | 它是你敲键盘写的那个东西 |
| **标记/样式语言** | 描述结构和外观，不是编程逻辑 | HTML, CSS, XML, Markdown | 它没有"执行逻辑"—你写了什么，页面就显示什么 |
| **查询语言** | 向数据库描述"要什么数据" | SQL, GraphQL, Cypher (Neo4j) | 你描述结果，不描述步骤 |
| **引擎 / 运行时** | 执行代码的底层程序 | V8 (JS), JVM (Java/Kotlin), CPython, Dart VM, .NET CLR, 浏览器渲染引擎 (Blink/WebKit/Gecko) | 它运行你的代码；你不是直接用它，而是通过语言或框架间接调用 |
| **平台 SDK** | 操作系统的原生开发工具包 | Android SDK, iOS SDK, Win32 API, Cocoa, Web APIs (DOM/Fetch) | 它给你调用系统能力的接口 |
| **UI 框架** | 组织界面代码的结构和渲染方案 | React, Vue, Angular, Svelte, Solid, SwiftUI, UIKit, Jetpack Compose | 它决定了你怎么组织界面和更新 DOM/View |
| **元框架** | UI 框架 + 路由 + SSR + 数据加载 + 构建整合 | Next.js, Nuxt, SvelteKit, Remix, Astro | 它在框架之上，一个命令包含"框架+构建+路由+服务端渲染" |
| **后端框架** | 组织后端代码的结构（路由、中间件、序列化、数据库连接） | Django, Spring Boot, ASP.NET Core, FastAPI, Express, Gin, Laravel, Rails | 它决定了你怎么写 API 和中间件 |
| **跨平台框架** | 写一份代码，编译到多个平台 | Flutter, React Native, Kotlin Multiplatform, Tauri, Electron | 一份代码 → 多平台产物 |
| **组件库** | 预制 UI 组件集合（按钮、对话框、表格），在 UI 框架之上 | shadcn/ui, Ant Design, MUI, Chakra UI | 它不是框架；它依赖某个框架（通常是 React/Vue），提供可直接使用的 UI 块 |
| **游戏引擎** | 渲染引擎 + 物理引擎 + 音频 + 脚本系统 + 编辑器 | Unity, Unreal Engine, Godot | 它不仅是运行时，还包括可视化编辑器和资源管线 |
| **包管理器** | 管理第三方依赖的下载、版本和安装 | npm, pip, Cargo, go mod, NuGet, Maven/Gradle | 你用 `install`/`add` 命令调用它 |
| **构建/打包工具** | 将源码编译、打包、优化为可分发产物 | Vite, Webpack, esbuild, CMake, Gradle, Xcode Build | 开发时写完代码，构建工具负责"从源码到产物" |
| **版本控制** | 追踪代码变更历史、协作的基础设施 | Git, GitHub, GitLab, Bitbucket | 你的代码放在哪、怎么追踪历史 |
| **CI/CD** | 自动化测试、构建、部署的管线 | GitHub Actions, GitLab CI, Jenkins, CircleCI | 推送代码后，自动触发的流水线 |
| **云平台** | 提供计算、存储、网络等基础设施的商业服务 | AWS, GCP, Azure, Vercel, Netlify, Fly.io, Railway | 不用买物理机器，按需租用计算资源 |
| **基础设施软件** | 独立部署的软件，你的应用依赖它运行 | Docker, K8s, Nginx, PostgreSQL, Redis, RabbitMQ, Prometheus | 你配置和运维它，不在它里面写业务代码 |

### 最容易混淆的几组区分

**语言 vs 框架**：

```
JavaScript = 语言（语法和语义规则）
React      = 框架（用 JavaScript 写的、帮你组织 UI 代码的工具）

你用 JavaScript 写 React 组件。React 本身是 JavaScript 代码。
```

**框架 vs 元框架**：

```
React    = UI 框架（只管"数据怎么映射到界面"）
Next.js  = 元框架（React + 路由 + SSR + 数据加载 + 构建，一口全包）

Next.js 依赖 React，在 React 之上添加了应用级的完整方案。
没有 React 就没有 Next.js。
```

**平台 SDK vs 框架**：

```
Android SDK = 谷歌提供的原生开发工具包（Java/Kotlin 调用系统 API）
Jetpack Compose = 谷歌在 Android SDK 之上提供的 UI 框架

Android SDK 给你调用相机、GPS、文件系统的能力。
Jetpack Compose 让你用声明式的方式写 Android 界面。
```

**跨平台框架 vs 原生平台 SDK**：

```
Android SDK + Kotlin = 只能在 Android 上跑
iOS SDK + Swift      = 只能在 iOS 上跑

Flutter (Dart) = 一份代码，编译为 Android App + iOS App + Web + 桌面
React Native (JS) = 一份代码，桥接到 Android 和 iOS 的原生 UI 组件
```

---

## 第三步：Web 生态——按角色展开

Web 开发的完整技术栈：

```
用户浏览器
    │
    ├─ HTML ── 页面的骨架（标题、段落、列表、表格）
    ├─ CSS  ── 页面的外观（颜色、字体、间距、布局）
    └─ JavaScript ── 页面的行为（响应用户操作、发请求、改 HTML/CSS）
         │
         ├─ TypeScript ── JavaScript + 静态类型（编译时检查类型错误）
         │
         ├─ UI 框架 ── 解决"状态变化后如何自动更新 DOM"
         │   ├─ React (Meta)
         │   ├─ Vue (社区驱动)
         │   ├─ Angular (Google)
         │   ├─ Svelte (社区驱动，编译时框架)
         │   └─ Solid (社区驱动，细粒度响应式)
         │
         ├─ CSS 方案 ── 解决"样式如何组织而不互相冲突"
         │   ├─ Tailwind CSS ── 工具类组合（当前主流）
         │   ├─ CSS Modules ── 编译时加哈希，自动作用域隔离
         │   └─ styled-components ── CSS 写在 JS 中（运行时方案）
         │
         ├─ 状态管理 ── 解决"不同组件如何共享数据"
         │   ├─ UI 状态：Zustand (React), Pinia (Vue)
         │   └─ 服务端缓存：TanStack Query (React), Vue Query
         │
         ├─ 元框架 ── 解决"完整应用需要的路由、SSR、构建"
         │   ├─ Next.js ── React 的元框架（Vercel）
         │   ├─ Nuxt ── Vue 的元框架
         │   ├─ SvelteKit ── Svelte 的元框架
         │   ├─ Remix ── React 的元框架（Shopify，Web 标准优先）
         │   └─ Astro ── 多框架兼容，侧重内容型站点
         │
         ├─ 组件库 ── 解决"不重写基础 UI 元素"
         │   ├─ shadcn/ui ── 源码复制到项目中（推荐）
         │   ├─ Ant Design ── 完整设计系统
         │   ├─ MUI ── Material Design 的 React 实现
         │   └─ Radix UI ── 无样式的行为组件（shadcn/ui 的基础）
         │
         ├─ 打包工具 ── 模块合并、TypeScript 编译、代码压缩、代码分割
         │   ├─ Vite ── 当前主流（ESM 原生 + esbuild + Rollup）
         │   ├─ Webpack ── 老牌，配置复杂但生态全
         │   ├─ esbuild ── Go 写，极快，常用于库而非应用
         │   ├─ Turbopack ── Rust 写，Next.js 内置
         │   └─ SWC ── Rust 写的 TS/JS 编译器（替代 Babel）
         │
         ├─ 测试工具
         │   ├─ Vitest / Jest ── 单元测试 + 组件测试
         │   ├─ Playwright / Cypress ── 浏览器 E2E 测试
         │   └─ Testing Library ── 以用户视角测试组件交互
         │
         └─ WebAssembly (Wasm) ── 在浏览器中运行非 JS 语言的高性能代码
             ├─ 编译目标：C/C++/Rust/Go → Wasm 字节码
             └─ 场景：图像处理、游戏引擎、加密、CAD 预览

        ┌─────────────────────────────────────────
        │
Web 后端：
        │
        ├─ 语言选型
        │   ├─ JavaScript/TypeScript ── Node.js 运行时
        │   ├─ Python
        │   ├─ Java / Kotlin ── JVM 运行时
        │   ├─ C# ── .NET 运行时
        │   ├─ Go ── 编译为独立二进制
        │   ├─ Rust ── 编译为独立二进制
        │   ├─ PHP
        │   └─ Ruby
        │
        ├─ 后端框架 ── 处理路由、中间件、序列化、数据库连接
        │   ├─ JS/TS: Express / Fastify / Hono
        │   ├─ Python: Django / FastAPI / Flask / Litestar
        │   ├─ Java / Kotlin: Spring Boot / Quarkus / Micronaut
        │   ├─ C#: ASP.NET Core
        │   ├─ Go: 标准库 net/http / Gin / Echo / Chi
        │   ├─ Rust: Axum / Actix-web / Rocket
        │   ├─ PHP: Laravel / Symfony
        │   └─ Ruby: Rails / Sinatra
        │
        ├─ API 范式 ── 前后端之间的约定格式
        │   ├─ REST ── 资源导向，HTTP 方法表示操作
        │   ├─ GraphQL ── 客户端指定需要的字段
        │   ├─ gRPC ── 二进制协议，服务间通信
        │   ├─ tRPC ── 全栈 TypeScript 端到端类型安全
        │   ├─ WebSocket ── 双向持久连接（聊天、协作编辑）
        │   └─ SSE ── 服务端向客户端单向推送（实时通知）
        │
        ├─ ORM / 数据库工具 ── 抽象 SQL 为对象操作（或从 SQL 生成类型安全代码）
        │   ├─ JS/TS: Prisma / Drizzle ORM / Knex.js
        │   ├─ Python: SQLAlchemy / Django ORM
        │   ├─ Java/Kotlin: Hibernate / jOOQ / Exposed
        │   ├─ Go: GORM / sqlc (类型安全 SQL 生成)
        │   ├─ Rust: Diesel / SQLx
        │   └─ C#: Entity Framework Core
        │
        ├─ 数据库
        │   ├─ 关系型：PostgreSQL (首选)、MySQL/MariaDB、SQLite
        │   ├─ 文档型：MongoDB、CouchDB
        │   ├─ 图数据库：Neo4j
        │   ├─ 时序数据库：TimescaleDB、InfluxDB
        │   └─ 内存数据：Redis (缓存/会话/限流/队列)
        │
        ├─ 认证 / 授权
        │   ├─ Auth0 / Clerk / Firebase Auth ── 托管式身份认证服务
        │   ├─ Keycloak / Ory ── 自部署开源方案
        │   ├─ Passport.js ── Node.js 认证中间件
        │   └─ OAuth 2.0 / OIDC ── 第三方登录的标准协议
        │
        └─ API 文档 / 规范
            ├─ Swagger / OpenAPI ── REST API 描述标准，可生成交互文档
            └─ GraphQL Schema ── GraphQL 的自文档化能力
```

---

## 第四步：移动端生态——按平台展开

移动端和 Web 最大的区别：**运行环境不是开放的浏览器，而是由苹果和谷歌严格控制的移动操作系统。**

### 原生开发（每个平台单独写代码）

| | iOS | Android |
|------|-----|---------|
| **语言** | Swift | Kotlin (之前是 Java) |
| **UI 框架** | SwiftUI (新) / UIKit (旧) | Jetpack Compose (新) / XML Views (旧) |
| **IDE** | Xcode (仅 macOS) | Android Studio (跨平台) |
| **SDK** | iOS SDK (Cocoa Touch) | Android SDK |
| **分发** | App Store (审核严格) | Google Play (审核较苹果宽松) |
| **特点** | 最完整调用系统能力 | 同上 |

**原生开发关系链**：

```
Swift (语言)
  └─ SwiftUI (苹果提供的 UI 框架，声明式)
       └─ iOS SDK (调用相机、GPS、推送通知等系统能力)
            └─ Xcode (IDE、模拟器、打包上传)

Kotlin (语言，由 JetBrains 开发，谷歌官方推荐)
  └─ Jetpack Compose (谷歌提供的 UI 框架，声明式)
       └─ Android SDK (调用系统能力)
            └─ Android Studio (IDE、模拟器、打包上传)
```

### 跨平台开发（写一份代码，编译到两端）

| 框架 | 语言 | 原理 | 定位 |
|------|------|------|------|
| **Flutter** | Dart (Google) | 自带渲染引擎 (Skia/Impeller)，不依赖系统 UI 组件。Dart → ARM 机器码 | 移动端跨平台首选 |
| **React Native** | JavaScript | 通过"桥"调用平台原生 UI 组件（Android → 原生 Button → React Native 翻译 → JS 收到事件） | Web 前端开发者迁移移动端的最短路径 |
| **Kotlin Multiplatform** | Kotlin | 业务逻辑跨平台共享，UI 仍各自写原生 | 已有 Android 应用，想共享代码到 iOS |
| **Expo** | JavaScript | React Native 的"元框架"——整合了构建、热更新、推送通知 | React Native 开发的标准起点 |

**移动端状态管理**：

| 框架 | 状态管理方案 |
|------|------------|
| Flutter | Riverpod (推荐)、Bloc、Provider (旧) |
| React Native | 同 Web——Zustand、TanStack Query |
| SwiftUI | `@State` / `@Observable` (内建)、SwiftData |
| Jetpack Compose | `remember` / `ViewModel` (内建) |

**移动端构建与依赖**：

| 平台 | 构建系统 | 依赖管理 |
|------|---------|---------|
| iOS | Xcode (xcodebuild) | Swift Package Manager (SPM) / CocoaPods (旧) |
| Android | Gradle | Gradle (Maven Central + Google Maven) |
| Flutter | flutter build (底层调 Gradle / Xcode) | pub.dev (Dart 注册中心) |

**Flutter 和 React Native 的区别**：

```
Flutter:
Dart 代码 ──[编译]──▶ ARM 机器码 ──▶ Flutter 引擎 ──▶ 自绘 UI
                                                      │
                                             不经过系统 UI 组件
                                             所有像素由 Flutter 自己画

React Native:
JavaScript 代码 ──▶ 桥 (Bridge) ──▶ 原生 UI 组件 (真正的 Android Button / iOS UIButton)
                                    │
                              UI 是系统原生的
                              JS 线程和原生线程通过桥异步通信
```

**Flutter 在移动端之外**：Flutter 也可以编译到 Web（Dart → JS）和桌面（Windows/macOS/Linux）。它不只是一个移动框架。

---

## 第五步：桌面端生态

| 方案 | 原理 | 适合 |
|------|------|------|
| **Electron** (JS/TS) | 内嵌 Chromium 浏览器 + Node.js。界面用 Web 技术写，系统能力通过 Node.js 调用。VSCode、Discord、Slack 用的是它 | Web 前端开发者做桌面应用的最短路径 |
| **Tauri** (Rust + JS/TS) | 用系统自带的 WebView（不嵌入 Chromium），后端用 Rust。包体积比 Electron 小 10 倍 | 新一代轻量方案 |
| **Qt** (C++ / Python) | 跨平台 C++ GUI 框架，最老牌。QML 用于声明式 UI | 需要原生性能和完全控制 |
| **SwiftUI** (macOS) | 苹果 macOS 原生 UI | 纯苹果生态 |
| **Flutter** | 同样支持桌面平台 | 一份代码覆盖移动+Web+桌面 |

---

## 第六步：游戏开发——一个独立的完整生态

游戏开发和"前端/后端"的 Web 开发模式完全不同——游戏引擎**同时管渲染、物理、音频、输入、网络、脚本和资源管线**，不区分"前端"和"后端"角色。

### 游戏引擎是什么

游戏引擎是把游戏开发中所有共性需求整合在一起的**一体化平台**：

```
┌──────────────────────────────────────────────┐
│                游戏引擎                        │
│                                              │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐      │
│  │ 渲染引擎  │ │ 物理引擎  │ │ 音频系统  │      │
│  │(3D/2D 显示)│ │(碰撞/重力)│ │(音效/音乐)│      │
│  └──────────┘ └──────────┘ └──────────┘      │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐      │
│  │ 脚本系统  │ │ 资源管线  │ │ 编辑器 UI │      │
│  │(逻辑/行为)│ │(模型/贴图)│ │(场景编辑器)│      │
│  └──────────┘ └──────────┘ └──────────┘      │
│  ┌──────────┐ ┌──────────────────────┐       │
│  │ 网络层    │ │ 构建与分发(多平台打包) │       │
│  └──────────┘ └──────────────────────┘       │
└──────────────────────────────────────────────┘
```

| 引擎 | 语言 | 定位 | 代表作品 |
|------|------|------|---------|
| **Unity** | C# | 通用引擎，移动端/独立游戏首选，市场占比最大 | Hollow Knight, Genshin Impact, Among Us |
| **Unreal Engine (UE5)** | C++ / Blueprint (可视化脚本) | 3A 级画质，开放世界，影视级渲染 | Fortnite, Black Myth: Wukong, Hellblade 2 |
| **Godot** | GDScript (类 Python) / C# / C++ | 开源、轻量、社区驱动 | Cassette Beasts, Brotato |
| **Bevy** | Rust | Rust ECS 架构的游戏引擎（还在早期） | 实验性项目 |
| **RPG Maker / Ren'Py** | 各自的 DSL | 特定类型（RPG / 视觉小说） | To the Moon, Doki Doki Literature Club |

### 游戏引擎和"框架"的区别

框架（React）只帮你组织界面代码。游戏引擎提供**完整的运行时**——渲染管线、物理世界模拟、音频播放、动画系统——外加**可视化编辑器**让你摆放场景、调节参数。它不是"库"也不是"框架"，更接近"工业套件"。

### 游戏后端（多人游戏）

单机游戏不需要"后端"——一切在玩家机器上。多人联机需要服务端：

- **Photon / PlayFab**：托管式多人游戏后端（房间管理、匹配、排行榜）
- **自建 C++/Go/Rust 游戏服务器**：需要处理实时网络同步、延迟补偿、反作弊

---

## 第七步：嵌入式 / IoT——资源受限的设备

当设备只有几百 KB 内存、几十 MHz CPU、没有操作系统时，开发方式完全不同。

| 设备层级 | 内存 | 操作系统 | 语言 | 框架/SDK |
|---------|------|---------|------|---------|
| 微控制器 (MCU) | KB 级 | 无 (裸金属) / FreeRTOS | C / C++ / Rust / Zig | Arduino, ESP-IDF, STM32 HAL |
| 嵌入式 Linux | MB-GB 级 | Linux (Buildroot/Yocto) | C / C++ / Rust / Go / Python | 标准 Linux API |
| 单板计算机 (SBC) | GB 级 | 完整 Linux (Raspberry Pi OS) | 任何语言 | 同桌面 Linux |

**嵌入式开发的关键区别**：
- **没有"前端"**——没有屏幕，或者只有简单的 LED/段码屏
- **没有"包管理器"**——依赖通常 vendoring 或通过 RTOS SDK 管理
- **交叉编译是常态**——在 PC 上编译，烧录到芯片上
- **调试工具不同**——JTAG/SWD 调试器、逻辑分析仪、串口打印

---

## 第八步：CLI / Serverless / Edge——运行形态不只有"服务器常驻"

除了"服务器持续运行"，还有几种不同的运行形态：

| 形态 | 说明 | 代表 |
|------|------|------|
| **CLI 工具** | 命令行下运行，执行完退出。不需要监听端口 | Git, ripgrep, fzf, systemctl |
| **Serverless (FaaS)** | 不需要管服务器。函数被事件触发运行，按调用计费 | AWS Lambda, Cloudflare Workers, Vercel Functions |
| **Edge Computing** | 代码跑在全球分布的边缘节点上，离用户最近 | Cloudflare Workers, Vercel Edge, Deno Deploy |
| **Cron / 定时任务** | 按预定时间执行 | systemd timer, cron, K8s CronJob |
| **流处理** | 持续消费数据流，实时计算 | Kafka Streams, Flink, RisingWave |

---

## 第九步：基础设施——所有平台的共同底座

不管你是 Web 后端、移动后端，还是桌面应用的服务端，底层基础设施是一样的：

```
                   [你的应用代码]
                        │
    ┌───────────────────┼───────────────────┐
    │                   │                   │
    ▼                   ▼                   ▼
┌────────┐    ┌──────────────┐    ┌──────────────┐
│ Nginx  │    │ 应用服务器    │    │ 应用服务器    │
│ 负载均衡│    │ (副本 1)     │    │ (副本 2)     │
│ 反向代理│    └──────┬───────┘    └──────┬───────┘
└────────┘           │                   │
                     └─────────┬─────────┘
                               │
              ┌────────────────┼────────────────┐
              │                │                │
              ▼                ▼                ▼
        ┌─────────┐    ┌──────────┐    ┌──────────┐
        │PostgreSQL│   │  Redis   │    │RabbitMQ  │
        │ 主数据库  │   │  缓存     │    │ 消息队列  │
        └─────────┘    └──────────┘    └──────────┘
```

### 按功能分类

**容器与编排**：

| 工具 | 是什么 | 解决什么问题 |
|------|--------|------------|
| **Docker** | 容器引擎 | "在我机器上能跑"——把应用和所有依赖打包在一起，任何地方行为一致 |
| **Kubernetes (K8s)** | 容器编排系统 | 当你需要几十上百个容器在多台机器上协同运行时，谁来分配资源、重启失败容器、滚动更新？ |

**理解 Docker 和 K8s 的关系**：Docker 是"一个容器怎么运行"，K8s 是"成千上万个容器怎么管理"。K8s 是 Docker 的上层——它调度 Docker 容器。

**Web 服务器与负载均衡**：

| 工具 | 是什么 | 解决什么问题 |
|------|--------|------------|
| **Nginx** | HTTP 服务器 / 反向代理 / 负载均衡 | 把用户请求分发给多台应用服务器；直接托管静态文件（前端构建产物） |

**数据存储**（详见 [后端开发全景指南](backend.md)）：

| 工具 | 类型 | 一句话 |
|------|------|--------|
| **PostgreSQL** | 关系型数据库 | 通用数据存储，首选 |
| **MySQL / MariaDB** | 关系型数据库 | 广泛部署的替代方案 |
| **SQLite** | 嵌入式数据库 | 移动端、Edge、小项目 |
| **MongoDB** | 文档数据库 | Schema 不固定的场景 |
| **Redis** | 内存数据存储 | 缓存、会话、限流、排行榜、队列——用途最多的组件 |

**消息队列**：

| 工具 | 模型 | 一句话 |
|------|------|--------|
| **RabbitMQ** | 传统消息代理 | 可靠投递，任务队列 |
| **Kafka** | 分布式日志 | 高吞吐、可回放，事件流 |
| **NATS** | 极简消息 | 低延迟、微服务通信、边缘计算 |

### CI/CD——自动化测试、构建、部署

当你把代码推送到 GitHub 之后，谁来做这些事：跑测试 → 构建 → 部署到服务器？CI/CD 管线自动完成这个流程。

| 工具 | 是什么 | 特点 |
|------|--------|------|
| **GitHub Actions** | GitHub 内置的 CI/CD | 事件驱动（push、PR、schedule），社区 actions 市场 |
| **GitLab CI** | GitLab 内置的 CI/CD | `.gitlab-ci.yml` 配置，与 GitLab 深度集成 |
| **Jenkins** | 老牌 CI 服务器 | 最灵活（插件生态庞大），但也最需要维护 |
| **CircleCI** | 云原生 CI/CD | 专注速度和并行 |

### 云平台——不用买物理机器的计算资源

云平台按需提供计算、存储、网络。按抽象层次从低到高：

| 层次 | 你管什么 | 云管什么 | 代表 |
|------|---------|---------|------|
| **IaaS**（基础设施即服务） | OS、运行时、应用代码 | 物理机、网络、存储 | AWS EC2、GCP Compute Engine |
| **PaaS**（平台即服务） | 应用代码 | OS、运行时、扩缩容 | Fly.io、Railway、Render、Heroku |
| **FaaS**（函数即服务） | 函数代码 | 一切 | AWS Lambda、Cloudflare Workers |
| **静态托管** | HTML/CSS/JS 文件 | 一切，连函数都不用写 | Vercel、Netlify、GitHub Pages |

**主要云厂商**：

| 厂商 | 一句话 | 特点 |
|------|--------|------|
| **AWS** | 云服务的百科全书 | 服务最多，市场第一，复杂度极高 |
| **GCP** | Google 云 | K8s（GKE）、BigQuery、AI 服务领先 |
| **Azure** | 微软云 | 与 .NET / Active Directory / Office 365 深度集成 |
| **Vercel** | 前端部署的最优体验 | 连 GitHub 仓库，推送即部署，Next.js 的娘家 |
| **Netlify** | 静态站点 + Serverless | 最早做 Jamstack 的平台 |
| **Fly.io** | 全球分布容器 | 把 Docker 容器部署到全球各地的边缘节点 |
| **Railway** | 极简部署 | `railway up`，自动检测语言和框架 |
| **Cloudflare** | CDN + Edge + DDoS | Workers（边缘计算）、Pages（静态托管）、R2（对象存储） |

### 可观测性——当系统变复杂后如何理解它

（详见 [后端开发全景指南](backend.md) 的第九层）

| 工具 | 回答的问题 | 一句话 |
|------|-----------|--------|
| **Prometheus** | "什么数值、多快、多少错？" | 拉取式指标收集，云原生事实标准 |
| **Grafana** | "怎么可视化这些数值？" | 对接 Prometheus/Loki/Tempo，画仪表盘和告警 |
| **Loki** | "日志里有什么？" | Grafana 配套的轻量日志聚合系统 |
| **OpenTelemetry** | "一个请求经过了哪些服务？" | 跨语言的可观测性采集标准——统一日志、指标、链路追踪 |
| **Jaeger / Tempo** | "调用链路上哪个环节慢了？" | 分布式链路追踪 |
| **ELK Stack** | "日志的全文搜索" | Elasticsearch（存储+搜索）+ Logstash（处理）+ Kibana（可视化） |

### 对象存储——文件不存服务器本地

当有多台应用服务器时，用户上传的文件不能存任何一台服务器的本地磁盘（下一请求可能被路由到另一台，文件不存在）。对象存储是**独立于服务器的文件存储**：

| 服务 | 是什么 | 一句话 |
|------|--------|------|
| **Amazon S3** | 对象存储行业标准 | 所有对象存储的 API 都兼容 S3 |
| **MinIO** | 自部署 S3 兼容存储 | 在自己服务器上跑 S3 兼容的对象存储 |
| **Cloudflare R2** | S3 兼容 + 零出站费 | 适合带宽密集型应用（图片、视频托管） |

### CDN——让用户更快地拿到静态资源

**CDN（Content Delivery Network）** 把你的静态文件（JS、CSS、图片、视频）缓存在全球各地的边缘服务器上。用户从离自己最近的节点获取文件——而不是从你唯一的服务器。

Cloudflare 是最大的 CDN 提供商，也提供 DDoS 防护和 Edge Workers。

### DNS——域名解析

用户输入 `example.com`，浏览器需要知道对应的 IP 地址。DNS 就是"域名 → IP"的查询系统。Cloudflare 的 `1.1.1.1` 和 Google 的 `8.8.8.8` 是公共 DNS 解析器。

### 机密管理——API 密钥和密码放哪里

数据库密码、第三方 API 密钥、证书——这些不能写在代码里，不能提交到 Git。

| 工具 | 一句话 |
|------|--------|
| **HashiCorp Vault** | 机密管理的行业标准——集中存储、访问控制、自动轮换 |
| **Doppler** | 面向开发者的机密管理——环境变量集中管理，多环境同步 |
| **GitHub Secrets** | 仓库级别的加密机密（用于 CI/CD） |
| **云厂商原生方案** | AWS Secrets Manager、GCP Secret Manager |

### 基础设施即代码（IaC）

手动在云控制台上点来点去创建服务器 → 不可复现、容易出错。IaC 用代码描述基础设施，可以版本控制和重复执行：

| 工具 | 写法 | 一句话 |
|------|------|--------|
| **Terraform** | HCL 声明式语言 | IaC 的行业标准——定义目标状态，自动推导执行计划 |
| **Pulumi** | TS/Python/Go/C# | 用真正的编程语言写基础设施 |

### 基础设施的层级总结

自底向上排列：

```
DNS（域名→IP 查询）
  │
CDN（全球边缘缓存静态文件）
  │
负载均衡（Nginx / Cloud LB）→ 分发到应用服务器
  │
应用服务器（Docker 容器）
  │
├─ 对象存储（S3 / MinIO）——文件不存本地
├─ 消息队列（RabbitMQ / Kafka）——异步处理
├─ 数据库（PostgreSQL / Redis）——持久化和缓存
├─ 机密管理（Vault / Doppler）——密钥集中管理
└─ 可观测性（Prometheus + Grafana）——监控、日志、追踪
  │
CI/CD（GitHub Actions）——自动测试、构建、部署
  │
基础设施即代码（Terraform）——管理以上所有
```

---

## 概念完整索引——文档中出现的每一个词

这个索引覆盖了本文提到的所有技术概念。按类别分组，每项标注"它是什么"和"它在哪个维度"。

### 编程语言

| 名称 | 类型 | 主要平台 | 一句话 |
|------|------|---------|--------|
| **JavaScript (JS)** | 动态、弱类型、JIT 编译 | Web 前端 + 后端 (Node.js) + 移动 (React Native) + 桌面 (Electron) | 唯一浏览器原生支持的语言，也是唯一能跑遍四个平台的语言 |
| **TypeScript (TS)** | JS 的超集，添加静态类型 | 同 JS | JS + 编译时类型检查，现代 Web 开发的默认选择 |
| **Python** | 动态、强类型、字节码 VM | 后端 + 数据科学 + AI/ML + 脚本 + 教育 | 通用脚本语言，生态覆盖最广 |
| **Java** | 静态、强类型、JVM 字节码 | 后端 (Spring) + Android | 企业级后端的事实标准，Write Once Run Anywhere |
| **Kotlin** | 静态、强类型、JVM 字节码 | Android 原生 + 后端 (JVM) | JetBrains 设计，谷歌推荐的 Android 首选语言 |
| **Dart** | 静态、强类型、AOT+JIT | 移动端 (Flutter) + Web + 桌面 | Google 开发，主要作为 Flutter 的载体语言 |
| **C#** | 静态、强类型、.NET CLR | 后端 (ASP.NET) + 桌面 (WinUI/WPF) + 游戏 (Unity) | 微软生态的核心语言 |
| **Go** | 静态、强类型、编译为原生码 | 后端 + CLI 工具 + 基础设施 | Google 开发，goroutine 并发模型是其杀手特性 |
| **Rust** | 静态、强类型、编译为原生码 | 后端 + 系统编程 + 嵌入式 + 工具链 (SWC, Turbopack) | 所有权系统实现无 GC 的内存安全 |
| **C / C++** | 静态、编译为原生码 | 系统编程 + 游戏引擎 + 嵌入式 + 高性能后端 | 零成本抽象，控制一切；C++ 是世界上最复杂的语言之一 |
| **PHP** | 动态、弱类型、字节码 VM | Web 后端 | 为 Web 而生，Laravel 让其现代化 |
| **Ruby** | 动态、强类型、字节码 VM | Web 后端 (Rails) | Rails 定义了 MVC Web 框架的模式 |
| **Swift** | 静态、强类型、编译为原生码 | iOS / macOS 原生 | Apple 开发，取代 Objective-C |
| **Zig** | 静态、强类型、编译为原生码 | 系统编程 + 嵌入式 | 类似 C 的控制力 + 编译时代码执行，可交叉编译 C 项目 |
| **GDScript** | 动态、类 Python | 游戏 (Godot) | Godot 引擎的内置脚本语言 |

### 标记 / 样式 / 查询语言

| 名称 | 类别 | 一句话 |
|------|------|--------|
| **HTML** | 标记语言 | 页面的骨架——标题、段落、列表、表格 |
| **CSS** | 样式语言 | 页面的外观——颜色、字体、间距、布局、动画 |
| **Markdown** | 标记语言 | 轻量标记，本文就是用 Markdown 写的 |
| **XML** | 标记语言 | 可扩展标记，Android 布局、SOAP、配置文件 |
| **SQL** | 查询语言 | 关系型数据库的通用语言——告诉数据库"要什么"，不描述"怎么取" |
| **GraphQL** | 查询语言 | Facebook 开发，客户端精确指定需要的字段 |
| **Cypher** | 查询语言 | Neo4j 图数据库的查询语言 |

### 引擎 / 运行时

| 名称 | 是什么 | 运行什么 |
|------|--------|---------|
| **V8** | Google 的 JS/WebAssembly 引擎 | Chrome, Node.js, Deno, Electron 的 JS 执行环境 |
| **JavaScriptCore (JSC)** | Apple 的 JS 引擎 | Safari, Bun 的 JS 执行环境 |
| **SpiderMonkey** | Mozilla 的 JS 引擎 | Firefox |
| **JVM (Java Virtual Machine)** | Java/Kotlin/Scala 的运行时 | 字节码 → JIT → 机器码 |
| **.NET CLR (Common Language Runtime)** | C#/F# 的运行时 | 类似于 JVM，微软生态 |
| **CPython** | Python 的参考实现 | Python 字节码 → 解释执行 |
| **Dart VM** | Dart 的运行时 | Dart → JIT（开发）/ AOT（生产） |
| **Blink / WebKit / Gecko** | 浏览器渲染引擎 | 将 HTML+CSS 转换为屏幕上的像素；各自包含 JS 引擎 |

### 平台 SDK

| 名称 | 平台 | 提供什么 |
|------|------|---------|
| **DOM / Web APIs** | Web | 浏览器暴露给 JS 的能力：操作页面元素、发 HTTP 请求、访问摄像头/地理位置 |
| **Android SDK** | Android | 谷歌提供的原生开发工具包——调用相机、GPS、蓝牙、文件系统 |
| **iOS SDK (Cocoa Touch)** | iOS | Apple 提供的原生开发工具包 |
| **Win32 API** | Windows | 微软 Windows 的原生 API |
| **Cocoa / AppKit** | macOS | Apple macOS 的原生 API |

### Web 前端：UI 框架

| 名称 | 出品方 | 核心机制 | 一句话 |
|------|--------|---------|--------|
| **React** | Meta | Virtual DOM + 单向数据流 + Hooks | 市场份额最大的 Web UI 框架 |
| **Vue** | 社区 (尤雨溪) | 响应式系统 + 模板 + Composition API | 设计精巧，学习曲线平缓 |
| **Angular** | Google | 完整 MVC + 依赖注入 + RxJS | 企业级方案，TypeScript 优先 |
| **Svelte** | 社区 (Rich Harris) | 编译时框架——构建时将组件编译为直接操作 DOM 的 JS | 没有运行时框架代码，包体积极小 |
| **Solid** | 社区 | 细粒度响应式，无 Virtual DOM | React 的 API 风格 + Svelte 的性能特性 |

### Web 前端：元框架

| 名称 | 基于 | 一句话 |
|------|------|--------|
| **Next.js** | React | Vercel 出品，文件系统路由 + RSC + SSR + 构建，React 生态标准 |
| **Nuxt** | Vue | Vue 的 Next.js 等价物 |
| **SvelteKit** | Svelte | Svelte 官方应用框架 |
| **Remix** | React | Shopify 出品，拥抱 Web 标准 (Fetch API, FormData) |
| **Astro** | 多框架兼容 | 默认零 JS，内容型站点首选（博客、文档、营销页） |
| **Expo** | React Native | React Native 的"元框架"——整合构建、热更新、推送通知 |

### Web 前端：其他核心工具

| 名称 | 类别 | 一句话 |
|------|------|--------|
| **Tailwind CSS** | CSS 方案 | 工具类组合——当前最主流的样式方案 |
| **CSS Modules** | CSS 方案 | 编译时加哈希，自动作用域隔离 |
| **styled-components** | CSS-in-JS | 样式写在 JS 组件中（运行时方案） |
| **Zustand** | 状态管理 (React) | 极简全局状态——一个函数创建一个 store |
| **Pinia** | 状态管理 (Vue) | Vue 官方状态管理 |
| **TanStack Query** | 服务端缓存 (React) | 自动缓存、后台刷新、乐观更新 |
| **shadcn/ui** | 组件库 | 源码直接复制到项目中——不是 npm 包，完全可控 |
| **Ant Design** | 组件库 | 完整设计系统，企业级 |
| **MUI** | 组件库 | Material Design 的 React 实现 |
| **Radix UI** | 无样式行为组件 | 只处理行为（展开、键盘导航），不管样式 |
| **Headless UI** | 无样式行为组件 | Tailwind 团队出品，与 Radix 同类 |
| **Vite** | 打包工具 | ESM + esbuild + Rollup，当前 JS/TS 打包的标准选择 |
| **Webpack** | 打包工具 | 老牌打包器，配置复杂但插件生态最全 |
| **esbuild** | 打包工具 | Go 语言编写，极快 |
| **Turbopack** | 打包工具 | Rust 编写，Next.js 内置的 Webpack 替代品 |
| **SWC** | 编译器 | Rust 写的 TS/JS 编译器，替代 Babel |
| **Vitest** | 测试框架 | Vite 原生的单元测试框架 |
| **Jest** | 测试框架 | Meta 出品，老牌 JS 测试框架 |
| **Playwright** | E2E 测试 | 微软出品，跨浏览器自动化测试 |
| **Cypress** | E2E 测试 | 实时重载、时间旅行调试的 E2E 测试 |
| **Testing Library** | 测试工具 | 以用户视角测试组件交互 |

### Web 后端：框架

| 名称 | 语言 | 一句话 |
|------|------|--------|
| **Django** | Python | "电池全包"——ORM、认证、管理后台、模板，全内置 |
| **FastAPI** | Python | 高性能异步、自动生成 OpenAPI 文档、类型提示驱动 |
| **Flask** | Python | 微框架——只提供路由和请求处理，其他自己选 |
| **Spring Boot** | Java/Kotlin | Java 生态的事实标准，依赖注入、自动配置 |
| **Quarkus** | Java | Kubernetes 原生，编译为原生二进制（GraalVM） |
| **Micronaut** | Java/Kotlin | 编译时依赖注入，无反射，启动快 |
| **ASP.NET Core** | C# | 微软的高性能后端框架 |
| **Express** | JS/TS (Node.js) | Node.js 最老的框架，中间件模式 |
| **Fastify** | JS/TS (Node.js) | 比 Express 更快，Schema 驱动 |
| **Hono** | JS/TS | 支持多运行时（Node.js, Deno, Bun, Cloudflare Workers） |
| **Gin** | Go | 高性能 HTTP 框架 |
| **Echo** | Go | 简约 HTTP 框架 |
| **Chi** | Go | 轻量级、与标准库兼容的路由器 |
| **Axum** | Rust | Tokio 团队出品，基于 tower 中间件生态 |
| **Actix-web** | Rust | Actor 模型驱动，性能极高 |
| **Rocket** | Rust | 宏驱动的声明式后端框架 |
| **Laravel** | PHP | PHP 最流行的全栈框架，Eloquent ORM + Blade 模板 |
| **Symfony** | PHP | 企业级 PHP 框架，组件可独立使用 |
| **Rails** | Ruby | MVC 模式的行业标杆——"约定优于配置" |

### Web 后端：数据库与 ORM

| 名称 | 类别 | 一句话 |
|------|------|--------|
| **PostgreSQL** | 关系型数据库 | 功能最全、扩展性最强——新项目的默认选择 |
| **MySQL / MariaDB** | 关系型数据库 | 被广泛使用，复制方案成熟 |
| **SQLite** | 嵌入式关系型数据库 | 单文件、零配置——移动端、Edge、小项目首选 |
| **MongoDB** | 文档数据库 | Schema 不固定，嵌套结构 |
| **Redis** | 内存数据存储 | 缓存 + 会话 + 限流 + 队列 + 排行榜——用途最多的基础设施组件 |
| **Neo4j** | 图数据库 | 社交关系、推荐系统 |
| **TimescaleDB** | 时序数据库 | PostgreSQL 扩展，用于监控指标、物联网数据 |
| **InfluxDB** | 时序数据库 | 专用时序存储 |
| **Prisma** | ORM (JS/TS) | 声明式 Schema → 生成类型安全客户端 |
| **Drizzle ORM** | ORM (JS/TS) | 轻量 ORM，SQL-like API，无代码生成 |
| **SQLAlchemy** | ORM (Python) | Python ORM 的事实标准 |
| **Hibernate** | ORM (Java) | JPA 标准实现 |
| **GORM** | ORM (Go) | Go 最流行的 ORM |
| **sqlc** | 代码生成 (Go) | 写 SQL → 生成类型安全 Go 代码 |
| **Diesel** | ORM (Rust) | Rust 的类型安全 ORM |
| **SQLx** | 异步 SQL (Rust) | 编译时检查 SQL 语法 |
| **Entity Framework Core** | ORM (C#) | .NET 的 ORM |

### API 范式

| 名称 | 是什么 | 适用场景 |
|------|--------|---------|
| **REST** | 资源导向的 API 风格，HTTP 方法表示操作 | 浏览器 ↔ 服务器，最通用 |
| **GraphQL** | Facebook 的查询语言，客户端指定需要的字段 | 复杂数据关系、多客户端（iOS 需要的数据和 Web 不同） |
| **gRPC** | Google 的高性能 RPC，Protocol Buffers 二进制序列化 | 微服务之间的内部通信 |
| **tRPC** | 全栈 TypeScript 的端到端类型安全 RPC | 全栈 TS 项目 |
| **WebSocket** | 双向持久连接的协议 | 聊天、协作编辑、实时通知 |
| **SSE (Server-Sent Events)** | 服务端向客户端单向推送 | 实时通知、股票行情 |

### API 文档与认证

| 名称 | 类别 | 一句话 |
|------|------|--------|
| **Swagger / OpenAPI** | API 描述标准 | REST API 的标准化文档格式，可生成交互文档和客户端 SDK |
| **Auth0** | 托管式认证服务 | "用 Google 登录"的托管实现 |
| **Clerk** | 托管式认证服务 | 面向 SaaS 应用的即用型认证 |
| **Keycloak** | 自部署认证 | 开源身份和访问管理 |
| **Ory** | 自部署认证 | 开源认证基础设施 |
| **Passport.js** | 认证中间件 | Node.js 的认证中间件集合 |

### 移动端：原生开发

| 名称 | 类别 | 平台 | 一句话 |
|------|------|------|--------|
| **Swift** | 语言 | iOS / macOS | Apple 开发，取代 Objective-C |
| **SwiftUI** | UI 框架 | iOS / macOS | Apple 的声明式 UI 框架（新） |
| **UIKit** | UI 框架 | iOS | iOS 的传统 UI 框架（仍广泛使用） |
| **Kotlin** | 语言 | Android | JetBrains 开发，谷歌推荐的 Android 首选 |
| **Jetpack Compose** | UI 框架 | Android | Google 的声明式 Android UI 框架（新） |
| **XML Views** | UI 框架 | Android | Android 的传统布局方式（仍广泛使用） |
| **Xcode** | IDE | iOS / macOS | Apple 的官方 IDE，含模拟器和打包上传工具 |
| **Android Studio** | IDE | Android | Google 的官方 IDE（基于 IntelliJ） |

### 移动端：跨平台开发

| 名称 | 语言 | 原理 | 一句话 |
|------|------|------|--------|
| **Flutter** | Dart | 自带 Skia/Impeller 渲染引擎，不依赖系统 UI 组件 | 移动端跨平台首选，也可编译到 Web 和桌面 |
| **React Native** | JS/TS | 通过"桥"调用原生 UI 组件 | Web 前端开发者迁移移动端的最短路径 |
| **Kotlin Multiplatform** | Kotlin | 业务逻辑共享，UI 各自写原生 | 已有 Android 应用，想共享代码到 iOS |
| **Expo** | JS/TS | React Native 的"元框架" | React Native 开发的标准起点 |

### 移动端：状态管理与构建

| 名称 | 类别 | 平台/框架 |
|------|------|----------|
| **Riverpod** | 状态管理 | Flutter（推荐） |
| **Bloc** | 状态管理 | Flutter |
| **SwiftData** | 持久化框架 | iOS / SwiftUI |
| **Gradle** | 构建系统 | Android |
| **Swift Package Manager (SPM)** | 依赖管理 | iOS / macOS |
| **CocoaPods** | 依赖管理（旧） | iOS |
| **pub.dev** | 包注册中心 | Dart / Flutter |

### 桌面端

| 名称 | 语言 | 原理 | 一句话 |
|------|------|------|--------|
| **Electron** | JS/TS | 内嵌 Chromium + Node.js | VSCode、Discord、Slack 的技术栈 |
| **Tauri** | Rust + JS/TS | 用系统自带 WebView（不嵌入 Chromium），后端 Rust | 比 Electron 轻 10 倍 |
| **Qt** | C++ / Python | 跨平台 C++ GUI 框架 | 最老牌的跨平台 GUI |
| **SwiftUI** | Swift | Apple 原生 UI | 纯苹果生态桌面应用 |

### 游戏引擎

| 名称 | 语言 | 一句话 | 代表作品 |
|------|------|--------|---------|
| **Unity** | C# | 通用引擎，移动端/独立游戏首选，市场占比最大 | Hollow Knight, Genshin Impact |
| **Unreal Engine (UE5)** | C++ / Blueprint | 3A 级画质，影视级渲染 | Fortnite, Black Myth: Wukong |
| **Godot** | GDScript / C# / C++ | 开源、轻量、社区驱动 | Cassette Beasts, Brotato |
| **Bevy** | Rust | Rust ECS 架构的游戏引擎 | 实验性项目 |
| **Photon** | 托管服务 | 多人游戏后端——房间管理、匹配 | — |
| **PlayFab** | 托管服务 | 微软的游戏后端服务 | — |

### 嵌入式 / IoT

| 名称 | 类别 | 一句话 |
|------|------|--------|
| **Arduino** | 平台 | 开源硬件 + 简易 IDE + Wiring 语言（类 C++） |
| **ESP-IDF** | SDK | Espressif 的 ESP32 芯片开发框架 |
| **FreeRTOS** | 实时操作系统 | 微控制器的轻量 RTOS |
| **STM32 HAL** | SDK | STMicroelectronics 的硬件抽象层 |
| **Buildroot / Yocto** | 构建系统 | 定制嵌入式 Linux 发行版 |

### 基础设施：容器与编排

| 名称 | 类别 | 一句话 |
|------|------|--------|
| **Docker** | 容器引擎 | "在我机器上能跑"——把应用和所有依赖打包在一起 |
| **Kubernetes (K8s)** | 容器编排 | 管理成千上万个容器在多台机器上的调度、扩缩容、滚动更新 |
| **Docker Compose** | 单机多容器 | 在一台机器上启动应用 + 数据库 + 缓存 + Worker |
| **Podman** | 容器引擎 | Docker 的无守护进程替代品 |

### 基础设施：Web 服务器与负载均衡

| 名称 | 类别 | 一句话 |
|------|------|--------|
| **Nginx** | HTTP 服务器 / 反向代理 | 静态文件托管 + 反向代理 + 负载均衡 |
| **HAProxy** | 负载均衡 | 高性能 TCP/HTTP 负载均衡器 |
| **Caddy** | HTTP 服务器 | 自动 HTTPS，配置简洁 |

### 基础设施：消息队列

| 名称 | 模型 | 一句话 |
|------|------|--------|
| **RabbitMQ** | 传统消息代理 | 灵活路由，可靠投递，任务队列 |
| **Kafka** | 分布式日志 | 高吞吐，可回放，事件流 |
| **NATS** | 极简消息 | 低延迟，微服务通信 |

### CI / CD

| 名称 | 类别 | 一句话 |
|------|------|--------|
| **GitHub Actions** | CI/CD 平台 | 与 GitHub 深度集成的事件驱动自动化 |
| **GitLab CI** | CI/CD 平台 | GitLab 内置的 CI/CD 管线 |
| **Jenkins** | CI/CD 平台 | 老牌 CI 服务器，插件生态庞大 |
| **CircleCI** | CI/CD 平台 | 云原生 CI/CD 平台 |

### 云平台

| 名称 | 类型 | 一句话 |
|------|------|--------|
| **AWS** | 公有云 | 亚马逊——云服务的百科全书 |
| **GCP** | 公有云 | Google 云——K8s、BigQuery、AI 服务 |
| **Azure** | 公有云 | 微软云——与 .NET / Active Directory 深度集成 |
| **Vercel** | 平台即服务 | 前端部署的最优体验——连 GitHub 仓库，推送即部署 |
| **Netlify** | 平台即服务 | 静态站点托管 + Serverless 函数 |
| **Fly.io** | 平台即服务 | 全球分布的容器平台 |
| **Railway** | 平台即服务 | 极简部署体验 |
| **Cloudflare** | CDN + Edge | 全球 CDN + Workers + DDoS 防护 |

### 可观测性

| 名称 | 类别 | 一句话 |
|------|------|--------|
| **Prometheus** | 指标收集 | 拉取式指标收集——云原生标准 |
| **Grafana** | 可视化仪表盘 | 对接 Prometheus / Loki / Tempo，画仪表盘 |
| **Loki** | 日志聚合 | Grafana 配套的日志系统 |
| **OpenTelemetry** | 可观测性标准 | 跨语言的链路追踪、指标、日志采集标准 |
| **Jaeger** | 链路追踪 | 分布式链路追踪 |
| **Tempo** | 链路追踪 | Grafana 配套的链路追踪 |
| **ELK Stack** | 日志 + 搜索 | Elasticsearch + Logstash + Kibana |

### 对象存储

| 名称 | 类别 | 一句话 |
|------|------|--------|
| **Amazon S3** | 对象存储 | 对象存储的行业标准 API |
| **MinIO** | 自部署对象存储 | S3 兼容的自部署方案 |
| **Cloudflare R2** | 对象存储 | 零出站费的 S3 兼容存储 |

### 其他基础设施

| 名称 | 类别 | 一句话 |
|------|------|--------|
| **Git / GitHub / GitLab** | 版本控制 | 代码存放、版本追踪、协作 |
| **HashiCorp Vault** | 机密管理 | API 密钥、数据库密码、证书的集中管理 |
| **Doppler** | 机密管理 | 面向开发者的机密管理服务 |
| **Terraform** | 基础设施即代码 | 声明式管理云资源 |
| **Pulumi** | 基础设施即代码 | 用编程语言（TS/Python/Go）管理云资源 |

### 你最初提到的 16 个概念的快速对照

这 16 个词是本文的起点。如果你想直接看每个词在全局中的位置：

| 你提到的 | 类别 | 维度 |
|---------|------|------|
| HTML | 标记语言 | Web 前端 |
| CSS | 样式语言 | Web 前端 |
| JavaScript | 编程语言 | Web 前端 + 后端 + 移动 + 桌面 |
| TypeScript | 编程语言 (JS 超集) | 同上 |
| React | UI 框架 | Web 前端 |
| Vue | UI 框架 | Web 前端 |
| Angular | UI 框架 | Web 前端 |
| Next.js | 元框架 | Web 前端（基于 React） |
| Java | 编程语言 | 后端 + Android |
| Kotlin | 编程语言 | Android + 后端 (JVM) |
| Python | 编程语言 | 后端 + 数据科学 + AI |
| Django | 后端框架 | Web 后端 (Python) |
| C# / ASP.NET | 语言 / 后端框架 | Web 后端 (.NET) |
| Flutter | 跨平台 UI 框架 | 移动 + Web + 桌面 |
| React Native | 跨平台 UI 框架 | 移动端 |
| Kubernetes | 容器编排系统 | 基础设施 |
| Redis | 内存数据存储 | 基础设施 |

---

## 选型指南——"我要做 X，该选什么"

### 路径 A：我要做一个 Web 应用，有登录、有数据操作

```
前端：Vite + React + Tailwind CSS ──▶ 后续可升级到 Next.js
后端：选你会的后端语言
  ├─ Python → FastAPI 或 Django
  ├─ JS/TS → Hono 或 Fastify
  ├─ Java/Kotlin → Spring Boot
  ├─ Go → 标准库 net/http 或 Gin
  └─ C# → ASP.NET Core
数据库：PostgreSQL
缓存：Redis（用到再加）
部署：Docker Compose → 后续规模大了上 K8s
```

### 路径 B：我要做一个移动应用，iPhone 和 Android 都要支持

```
方案 1 (跨平台)：Flutter
  语言：Dart
  一份代码 → iOS + Android + (可选 Web/桌面)
  适合：大多数移动应用

方案 2 (跨平台，会用 React)：React Native + Expo
  语言：JS/TS
  学会了就同时会移动端和 Web 前端
  适合：Web 前端开发者转型

方案 3 (原生，追求极致)：SwiftUI (iOS) + Jetpack Compose (Android)
  需要写两套代码
  适合：大型 App，需要深度调用系统新特性
```

### 路径 C：我要做一个桌面应用

```
用过 Web 前端：Electron (VSCode 的技术栈) 或 Tauri (更轻量)
用过 Flutter：Flutter Desktop
追求原生性能：Qt (C++) 或 SwiftUI (仅 macOS)
```

### 路径 D：我是后端工程师，选语言

```
创业 / 快速验证：Python (Django/FastAPI) 或 JS/TS (Node.js)
高性能 / 微服务：Go 或 Rust
企业级 / 大型团队：Java (Spring Boot) 或 C# (ASP.NET)
系统编程 / 嵌入式：C / C++ / Rust
```

### 路径 E：我想做游戏

```
独立小游戏 / 移动端游戏 → Unity + C#
  最成熟的生态，学习资源最多，市场占比最大

3A 级 / 高端画面 → Unreal Engine 5 + C++ / Blueprint
  影视级渲染，但学习曲线陡峭

开源 / 轻量 → Godot + GDScript (类 Python)
  完全免费开源，社区活跃

Rust 爱好者 → Bevy (Rust)
  仍在早期，适合实验性项目

多人游戏后端 → Photon / PlayFab（托管），或自建 C++/Go/Rust 游戏服务器
```

### 路径 F：我想做嵌入式 / IoT

```
Arduino → 入门首选。C++ 简化版，IDE 开箱即用

ESP32 / STM32 裸金属 → C + ESP-IDF / STM32 HAL
  没有操作系统，KB 级内存，交叉编译

嵌入式 Linux（Raspberry Pi 等）→ C / Rust / Go / Python
  有完整 Linux，开发体验接近桌面

实时操作系统 → FreeRTOS / Zephyr
  需要实时保证（毫秒级精确响应）
```

### 路径 G：我要写 CLI 工具

```
简单脚本 → Python 或 Shell
高性能 / 单文件分发 → Go 或 Rust
JS 生态 → Node.js + Commander.js
需要在 npm 上发布 → Node.js（npm 生态天然分发）
```

### 路径 H：我想用 Serverless / Edge

```
前端为主，需要少量后端逻辑 → Vercel Functions / Netlify Functions
全球低延迟、边缘节点运行 → Cloudflare Workers
AWS 生态 → AWS Lambda + API Gateway
Event-driven 数据处理 → AWS Lambda / GCP Cloud Functions
```

---

## 完整的开发世界地图

把前面所有内容收敛为一张完整图：

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                              基础设施（所有平台共享）                           │
│                                                                              │
│  云平台: AWS / GCP / Azure / Vercel / Fly.io / Cloudflare                    │
│  容器: Docker → K8s / Docker Compose                                         │
│  数据库: PostgreSQL / MySQL / SQLite / MongoDB / Redis / Neo4j               │
│  消息队列: RabbitMQ / Kafka / NATS                                           │
│  可观测性: Prometheus + Grafana + Loki / OpenTelemetry + Jaeger              │
│  存储/CDN: S3 / MinIO / Cloudflare CDN                                       │
│  CI/CD: GitHub Actions / GitLab CI / Jenkins                                 │
│  DNS / 机密管理 / IaC                                                        │
└───────────────────────────────┬──────────────────────────────────────────────┘
                                │
    ┌───────────────┬───────────┼───────────┬───────────────┬──────────────┐
    │               │           │           │               │              │
    ▼               ▼           ▼           ▼               ▼              ▼
┌────────┐  ┌──────────┐  ┌────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
│Web 前端 │  │ 移动端    │  │桌面端   │  │  游戏     │  │ 嵌入式    │  │ CLI /    │
│        │  │          │  │        │  │          │  │          │  │ Serverless│
│HTML/CSS│  │原生:     │  │Electron│  │引擎:     │  │MCU: C/Rust│  │Git, ripgrep
│JS/TS   │  │Swift/SwUI│  │Tauri   │  │Unity(C#)│  │+ FreeRTOS │  │systemctl  │
│        │  │Kotlin/   │  │Qt      │  │Unreal(C++)│ │Arduino   │  │           │
│React   │  │Compose   │  │SwiftUI │  │Godot    │  │          │  │Lambda     │
│Vue     │  │          │  │Flutter │  │Bevy(Rust)│ │Linux: Rust│  │Workers    │
│Svelte  │  │跨平台:   │  │        │  │          │  │+ Buildroot│  │Edge Fn    │
│Solid   │  │Flutter   │  │        │  │后端:     │  │          │  │           │
│        │  │React Ntv │  │        │  │Photon   │  │          │  │流处理:    │
│元框架: │  │Kotlin MP │  │        │  │PlayFab  │  │          │  │Kafka Strms│
│Next.js │  │          │  │        │  │          │  │          │  │Flink      │
│Nuxt    │  │          │  │        │  │          │  │          │  │           │
│SvelteKit│ │          │  │        │  │          │  │          │  │           │
│Astro   │  │          │  │        │  │          │  │          │  │           │
│        │  │          │  │        │  │          │  │          │  │           │
│样式:   │  │          │  │        │  │          │  │          │  │           │
│Tailwind│  │          │  │        │  │          │  │          │  │           │
│CSS Mod │  │          │  │        │  │          │  │          │  │           │
│        │  │          │  │        │  │          │  │          │  │           │
│组件库: │  │          │  │        │  │          │  │          │  │           │
│shadcn  │  │          │  │        │  │          │  │          │  │           │
│AntD/MUI│  │          │  │        │  │          │  │          │  │           │
└───┬────┘  └─────┬────┘  └───┬────┘  └─────┬────┘  └──────────┘  └──────────┘
    │              │           │              │
    └──────────────┼───────────┼──────────────┘
                   │           │
                   ▼           ▼  (HTTP / WebSocket / gRPC)
            ┌─────────────────────────────────────┐
            │          Web 后端（API 服务）          │
            │                                     │
            │  JS/TS: Node.js + Express/Hono      │
            │  Python: Django / FastAPI / Flask   │
            │  Java/Kotlin: Spring Boot / Quarkus │
            │  C#: ASP.NET Core                   │
            │  Go: 标准库 / Gin / Chi              │
            │  Rust: Axum / Actix-web             │
            │  PHP: Laravel                       │
            │  Ruby: Rails                        │
            │                                     │
            │  API 范式: REST / GraphQL / gRPC     │
            │  认证: JWT / OAuth 2.0 / Auth0       │
            │  ORM: Prisma / SQLAlchemy / sqlc    │
            └─────────────────┬───────────────────┘
                              │
                              ▼
            ┌─────────────────────────────────────┐
            │            数据存储层                  │
            │  PostgreSQL / MySQL / SQLite         │
            │  Redis / MongoDB / Neo4j             │
            │  InfluxDB / TimescaleDB              │
            └─────────────────────────────────────┘
```

**从这张图你可以直接回答**：

- "React 和 Django 是什么关系？" → React 是 Web 前端 UI 框架 (JS)，Django 是 Web 后端框架 (Python)。它们通过网络通信，可以配对使用。
- "Flutter 和 React Native 是什么关系？" → 都是跨平台移动框架，同一生态位上的竞争方案。Flutter 用 Dart 自绘 UI，React Native 用 JS 桥接原生 UI。
- "Kotlin 和 Java 是什么关系？" → 都跑在 JVM 上，都用于 Android 和后端。Kotlin 是 JetBrains 设计的更现代的语言，谷歌推荐它作为 Android 首选。
- "Next.js 和 React 是什么关系？" → Next.js 依赖 React。React 是 UI 框架，Next.js 是元框架——在 React 之上加了路由、SSR、构建。
- "Kubernetes 和 Docker 是什么关系？" → Docker 管单个容器，K8s 管成千上万个容器在多台机器上的编排。
- "Unity 和 React 是什么关系？" → 没有关系。Unity 是游戏引擎（渲染+物理+音频+编辑器），React 是 Web UI 框架。它们解决完全不同的问题。
- "Flutter 和 Unity 都是"跨平台"，区别是什么？" → Flutter 做界面应用（App），Unity 做可交互的实时 3D/2D 世界。Flutter 输出的是"按钮+列表+输入框"，Unity 输出的是"场景+角色+物理"。两者都是跨平台但领域不同。
- "嵌入式开发和后端开发用什么语言都行吗？" → 不。嵌入式（MCU 级）基本只能用 C/C++/Rust/Zig——没有操作系统，没有运行时，KB 级内存。后端可以选择几乎所有语言。

---

## 实用学习路线

1. **先定你要做什么平台**（Web？移动？桌面？游戏？嵌入式？后端？全栈？）
2. **学该平台的核心语言**（Web = JS/TS，移动 = Dart 或 Kotlin，后端 = 任一后端语言，游戏 = C# 或 C++，嵌入式 = C）
3. **学该平台的主流框架/引擎**（Web 前端 = React 或 Vue，后端 = 对应语言的框架，游戏 = Unity 或 Godot）
4. **做一个完整的项目**——前端+后端+数据库，部署上线（或：一个可玩的游戏原型，一个能用的命令行工具）
5. **遇到瓶颈再引入新组件**（缓存→Redis，容器→Docker，编排→K8s）

不要试图"先学完所有框架再开始做"。学完一个语言+一个框架后，立刻动手做项目。其他东西（Redis、K8s、队列）在项目需要的时候再学——你会有真实的动机和上下文。

---

## 延伸阅读

- [开发生态现状报告（2026）](ecosystem-state.md) — 全景图告诉你"每个东西是什么"，现状报告告诉你"现在谁主导、谁在上升、我该学什么"
