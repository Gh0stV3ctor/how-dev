# 部署全景——从构建产物到可访问的网站

## 什么是部署

部署 = 把构建产物放到一台持续开机、联网的服务器上，让它开始处理请求。

三件必须做的事：

1. **上传**：把文件/二进制传送到服务器上
2. **启动**：在服务器上运行你的程序（并确保它挂了能重启）
3. **联网**：让外部网络能访问到它（DNS、端口、防火墙）

---

## 前端部署

前端构建产物是**纯静态文件**——HTML + JS + CSS + 图片。它们不需要"运行"——只需要被**返回给浏览器**。

### 方式 1：用 Nginx 托管（最传统）

```
步骤：
1. 在本地构建：
   npm run build → .next/static/ 或 dist/

2. 上传到服务器：
   scp -r dist/* user@203.0.113.5:/var/www/frontend/

3. 配置 Nginx（已在服务器上跑着）：
   server {
       listen 443 ssl;
       server_name example.com;

       location / {
           root /var/www/frontend;
           try_files $uri /index.html;
       }
   }

4. 验证：
   浏览器访问 https://example.com → 看到你的页面

更新时：重复步骤 1-2
```

**`try_files $uri /index.html` 是什么**：
- 浏览器访问 `example.com/settings` 时，服务器上没有 `settings.html`
- `try_files` 告诉 Nginx：先找 `$uri` 对应的文件，找不到就把 `index.html` 返回
- 浏览器收到 `index.html`，React Router 接管 URL `/settings`，在客户端渲染设置页面
- 这就是 SPA（单页应用）路由的基础

### 方式 2：用静态托管平台（最简单）

| 平台 | 如何部署 | 额外提供 |
|------|---------|---------|
| **Vercel** | `git push` → 自动检测 Next.js → 构建 + 部署 | CDN、HTTPS 自动、Serverless Functions |
| **Netlify** | 同上 | 表单处理、身份认证、CDN |
| **Cloudflare Pages** | `git push` 或直接上传 | CDN（全球 330+ 节点）、Workers（在 CDN 节点跑代码） |
| **GitHub Pages** | `git push` 到特定分支 | 纯静态，免费，适合文档和个人页 |

### 方式 3：用 CDN + 自有服务器

```
构建产物上传到 S3（AWS 对象存储）
    │
    ▼
CloudFront（AWS CDN）从 S3 拉取文件
    │
    ▼
DNS 把 example.com 指向 CloudFront
    │
    ▼
用户访问时，从距离最近的 CDN 节点获取文件
```

**CDN（Content Delivery Network）是什么**：
- 全球分布的文件缓存服务器网络
- 你的静态文件（JS、CSS、图片）被复制到全球各地的 CDN 节点
- 用户从最近的节点下载——美国用户从美国节点获取，亚洲用户从亚洲节点获取
- 比从你的唯一服务器上获取快得多（物理距离导致延迟）

---

---

## 为什么要把前后端分开部署

你的直觉是对的：**一台服务器同时放前端静态文件和后端进程，完全可行。** Nginx 返回前端静态文件 + 反向代理到后端进程，这是生产环境中最常见的模式（见 [生产场景模式——模式 1](06-production-patterns.md)）。

分开部署不是因为"技术上必须"，而是因为**三个工程理由**。

### 理由 1：CDN 的物理距离优势

静态文件（JS/CSS/图片）体积大——几百 KB 到几 MB。API 响应体积小——几 KB。它们的**传输需求完全不同**。

```
单服务器部署（所有用户访问同一台机器）：
  东京用户 ──300ms──▶ 你的服务器（新加坡）
  伦敦用户 ──280ms──▶ 你的服务器（新加坡）
  前端文件（500KB）: 300ms 延迟 + 传输时间 = 很慢
  API 请求（2KB）:   300ms 延迟 + 传输时间 = 可接受

前端放 CDN（静态文件在全球多个节点）：
  东京用户 ──5ms──▶ CDN 东京节点（拿到 JS/CSS/图片）
              │
              └──300ms──▶ 你的服务器（新加坡）（只有 API 请求到这里）

  伦敦用户 ──5ms──▶ CDN 伦敦节点（拿到 JS/CSS/图片）
              │
              └──280ms──▶ 你的服务器（新加坡）（只有 API 请求到这里）
```

**前端文件离用户近（CDN），后端进程放在你控制的服务器上。** 这是物理延迟决定的，不是架构审美。

### 理由 2：故障隔离

```
同一台服务器：
  后端 Go 进程崩溃 → Nginx 还在 → 前端还能加载（只是 API 不可用）
  后端死循环吃满 CPU → Nginx 也被拖慢 → 前端也加载不出来
  服务器宕机 → 一切都没了

前端放在 CDN/Vercel，后端在自己服务器：
  后端崩溃 → 前端不受影响 → 用户至少看到"加载失败，请重试"
  服务器宕机 → 前端仍然能加载（只是功能不可用）
```

前端是用户看到的第一个东西。如果它因为后端问题而加载不出来，用户不会说"后端坏了"，他们会说"网站挂了"。

### 理由 3：部署节奏不同

```
前端：一天可能改 10 次（调 CSS、改文案、修 UI bug）
后端：一周可能改 1 次（数据库迁移、权限逻辑变更）

同一台服务器 → 每次前端修改都要重启服务 → 后端连接中断
分开部署     → 前端在 Vercel 上部署，后端在服务器上完全不受影响
```

### "分开部署"具体是什么

不是说"前端在北京、后端在新加坡"。最常见的"分开部署"：

```
前端静态文件：
  放在 Vercel / Cloudflare Pages / Netlify 上
  → 这些平台自动把文件复制到全球 CDN 节点
  → 配置 DNS：example.com → Vercel 的 CDN

后端进程：
  放在你自己的 VPS 上（或 AWS / Fly.io）
  → 暴露一个子域名：api.example.com → 你的服务器 IP
  → 前端代码里写：fetch("https://api.example.com/users")

用户访问 example.com → DNS 解析到 Vercel CDN → 下载 HTML/JS/CSS
  → JS 执行 → fetch("api.example.com/users") → DNS 解析到你的服务器
  → Nginx → 后端进程 → 返回 JSON
```

前端和后端在不同域名下，但浏览器不关心——它只关心"这个 URL 返回了什么数据"。

### 你该怎么做

对个人项目、学习项目：**一台服务器全搞定。** 不要为了"最佳实践"把简单事情复杂化。当以下症状出现时再考虑分开：

- 用户说"网站打开好慢"（前端文件下载是瓶颈）
- 后端崩溃导致整个网站打不开（需要故障隔离）
- 前端频繁更新导致后端反复重启（需要独立部署节奏）

在那之前，一个 VPS + Nginx 返回静态文件 + 反向代理到后端进程——完全足够。

## 后端部署

后端不是静态文件——是需要**持续运行**的进程。

### 方式 1：直接部署——最原始也最直接

```
步骤：
1. 编译：
   本地或 CI 上：go build -o server .

2. 上传：
   scp server user@203.0.113.5:/opt/my-app/server

3. 在服务器上创建 systemd service：
   /etc/systemd/system/my-app.service:
   [Unit]
   Description=My Application
   After=network.target

   [Service]
   Type=simple
   User=app
   WorkingDirectory=/opt/my-app
   ExecStart=/opt/my-app/server
   Restart=always           ← 进程崩溃后自动重启
   RestartSec=5

   [Install]
   WantedBy=multi-user.target

4. 启动：
   systemctl daemon-reload
   systemctl enable --now my-app
   → 进程在后台运行，监听 localhost:3001

5. Nginx 转发：
   location /api/ {
       proxy_pass http://localhost:3001;
   }

更新时：
   scp 新 binary → systemctl restart my-app
```

**为什么需要 systemd 而不是直接 `./server &`**：
- `./server &` 在后台运行，但你退出 shell 后它被杀了（SIGHUP）
- 崩溃了不会自动重启
- 开机不会自动启动
- systemd 和其他现代的进程管理器（PM2）解决这三个问题

### 方式 2：Docker 容器化部署——标准化打包

**Docker 解决什么问题**：你的开发环境可能和服务器不同（你 Mac 上写的 Go 二进制，服务器是 Linux）。Docker 把"应用 + 依赖 + 运行时"打包成一个标准化的镜像，在任何支持 Docker 的服务器上都能一模一样的运行。

```
步骤：

1. 写 Dockerfile（应用的"打包说明书"）：

   FROM golang:1.23-alpine AS builder
   WORKDIR /app
   COPY go.mod go.sum ./
   RUN go mod download
   COPY . .
   RUN CGO_ENABLED=0 go build -o server .

   FROM alpine:3.20
   RUN apk add --no-cache ca-certificates
   COPY --from=builder /app/server /server
   EXPOSE 3001
   CMD ["/server"]

2. 构建镜像：
   docker build -t my-app:v1.0.0 .

3. 上传到容器注册中心：
   docker tag my-app:v1.0.0 yourname/my-app:v1.0.0
   docker push yourname/my-app:v1.0.0
   → 镜像存在 Docker Hub（或其他 registry）

4. 在服务器上拉取并运行：
   docker pull yourname/my-app:v1.0.0
   docker run -d --name my-app -p 127.0.0.1:3001:3001 yourname/my-app:v1.0.0

   -d         = 后台运行
   --name     = 给容器取个名字（方便管理）
   -p 127.0.0.1:3001:3001 = 只允许本机访问 3001 端口，映射到容器的 3001
   127.0.0.1:3001:3001    = 关键——只绑定 localhost，不暴露给公网

5. Nginx 转发（同上）
```

### 方式 3：PaaS（Platform as a Service）——你只提供代码

```
Fly.io:
  fly launch  → 自动检测 Dockerfile → 构建镜像 → 部署到全球节点
  fly deploy  → 推送新版本 → 滚动更新（零停机）

Railway:
  git push → 自动检测 Go 项目 → 构建 → 部署 → 提供 HTTPS URL

Render:
  连 GitHub 仓库 → 选择"Go" → 自动构建 + 部署
```

**PaaS 和"自己管服务器"的本质区别**：

| | 自己管服务器 | PaaS |
|---|---|---|
| 你需要做 | 安装 OS → 装依赖 → 配 Nginx → 配 systemd → 配置防火墙 → 配 SSL 证书 | 推送代码 |
| 你得到 | 完全控制 | 零运维 |
| 你失去 | — | 灵活性（不能自选 Nginx 配置、某些语言/版本不支持） |

### 方式 4：Serverless——你只提供函数，平台负责跑

```
Vercel Serverless Functions:
  src/app/api/users/route.ts
    └── export async function GET(request: Request) { ... }

  部署后：
  有人请求 /api/users → Vercel 启动一个函数实例 → 执行你的代码 → 返回响应 → 实例销毁
  没有请求时 → 没有实例在运行 → 不花钱
  并发高时 → Vercel 自动启动更多实例

AWS Lambda:
  同一个函数概念，但更底层：
  写函数 → 上传到 Lambda → 设置触发器（API Gateway → 函数）
```

**Serverless 的关键约束**：
- 函数执行有时间上限（Vercel: 10s-60s, Lambda: 15min）
- 不能长驻后台进程（没有"永远在运行"的进程）
- 本地文件系统不可靠（每次请求可能是不同实例，文件不共享）
- **适合**：API 端点、数据处理、定时任务
- **不适合**：WebSocket 长连接、长时间计算、需要本地文件缓存的服务

---

## Docker 在什么位置——它是一个什么"层"

Docker 不是架构中的新层。在运行时全景图中，它的位置是：

```
没有 Docker：
  [物理服务器] → [操作系统] → [Nginx 进程] → [你的后端进程]

有了 Docker：
  [物理服务器] → [操作系统] → [Docker 引擎] → [容器] → [你的后端进程]
                                               (隔离环境)
```

**Docker 改变的是"打包和运行方式"，不是架构拓扑。** Nginx 还是那个 Nginx，PostgreSQL 还是那个 PostgreSQL——只是它们现在跑在容器里，而不是直接在宿主机上。

容器化的好处：
- **一致性**：你本地 Docker 环境和服务器 Docker 环境一模一样
- **隔离**：一个容器一个应用，配置不会互相污染
- **便携**：构建一次镜像，任何支持 Docker 的地方都能跑

---

## CI/CD 流水线

上面每一步都要手动执行吗？CI/CD 自动化了从"Git push"到"部署到服务器"的全部过程。

```
你：git push

GitHub Actions 自动执行（.github/workflows/deploy.yml）：

1. 检出代码
2. 安装依赖
3. 运行测试（go test / npm test）
4. 构建（go build / npm run build）
5. 构建 Docker 镜像
6. 推送到 Docker Hub
7. SSH 到服务器 → docker pull → docker-compose up -d
8. 健康检查（curl https://example.com/health → 确认 200 OK）

如果任何一步失败 → 通知你 → 部署不继续
```

---

## 环境变量——开发/预发布/生产怎么隔离

你的应用在不同环境需要不同的配置：

```bash
# 开发环境（你的电脑）
DATABASE_URL=postgresql://localhost:5432/myapp_dev
API_KEY=dev-key-123

# 预发布环境（测试服务器）
DATABASE_URL=postgresql://db-staging.internal:5432/myapp_staging
API_KEY=staging-key-456
STRIPE_SECRET_KEY=sk_test_...

# 生产环境（正式服务器）
DATABASE_URL=postgresql://db-prod.internal:5432/myapp
API_KEY=prod-key-789
STRIPE_SECRET_KEY=sk_live_...
```

**绝对不能**把生产环境的密钥写在代码里（提交到 Git）。密钥通过环境变量注入——服务器上的 systemd service 文件设置 `Environment=DATABASE_URL=...`，或者 Docker 用 `--env DATABASE_URL=...`。

---

## 部署策略——怎么上线新版本而不中断服务

### 滚动更新（Rolling Update）

```
旧版本实例 A ──▶ 停止 A → 启动新 A ──▶
旧版本实例 B ────────────────▶ 停止 B → 启动新 B ──▶
旧版本实例 C ────────────────────────────────▶ 停止 C → 启动新 C

全程都有实例在运行，用户不会感受到中断。
```

### 蓝绿部署（Blue-Green）

```
当前（蓝）：10 个实例，v1.0
新版（绿）：10 个实例，v1.1

切换：把负载均衡器的流量从蓝指向绿
如果 v1.1 有问题：把流量切回蓝（秒级回滚）
```

### 金丝雀发布（Canary）

```
90% 流量 → v1.0（当前版本）
10% 流量 → v1.1（新版，金丝雀）

监控 10% 用户的错误率、响应时间。
没问题 → 逐步增加 v1.1 的流量比例。
有问题 → 把金丝雀的流量切回 v1.0。
```

---

## 部署的各层抽象——从最多控制到最少维护

```
最多控制 / 最多工作                最少控制 / 最少工作
←──────────────────────────────────────────────────→

裸金属服务器      VPS          PaaS         Serverless     静态托管
(买物理机)    (租虚拟机)   (Fly.io/Railway)  (Vercel Fn)    (GitHub Pages)
    │              │              │               │              │
管 OS/网络      管 OS      管应用配置       管代码        管文件
管 Docker       管 Docker    不管 OS        不管 OS        不管 OS
管应用进程      管应用进程   不管 Docker     不管 Docker    不管进程
```

**你的项目在哪个抽象层级，取决于你的团队规模和运维能力。** Cal.diy 用 Turborepo Monorepo + Vercel（PaaS 层），Plane 用 Docker Compose 单机部署（VPS 层），都是合理的选择。

---

## 下一章

代码跑起来了，全世界能访问了。但每一层的技术选型（框架、语言、数据库、部署平台）是怎么决定的？

→ [05 技术选型](05-tech-selection.md)
