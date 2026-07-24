# 生态基础设施——那些"隐形"服务在做什么

前面六篇讲的是你自己写的代码和运行的服务器。但在你写代码的过程中，有一整套**你不直接管理的、但你的开发流程完全依赖的服务**。

本文讲这些"隐形"的服务是怎么运作的——不是教你用它们（你已经在用了），而是让你理解它们在你写的第一段代码和上线之间的精确位置。

---

## npm / PyPI / crates.io——包注册中心的运行机制

### 你 `npm install react` 时发生了什么

```
你的电脑
    │
    │  npm install react
    ▼
┌─────────────┐     HTTPS GET        ┌──────────────────────┐
│ npm CLI     │ ──────────────────▶  │ registry.npmjs.org   │
│ (本地工具)   │                      │ (npm 官方注册中心)     │
│             │ ◀──────────────────  │                      │
│             │    返回包元数据        │  存储了 200 万+ 包    │
└─────────────┘                      └──────────────────────┘
    │
    │ npm 根据元数据找到包的 tarball URL
    ▼
┌─────────────┐     HTTPS GET        ┌──────────────────────┐
│ npm CLI     │ ──────────────────▶  │ registry.npmjs.org   │
│             │                      │ /react/-/react-18.2.0.tgz │
│             │ ◀──────────────────  │                      │
│             │    返回 tarball       │                      │
└─────────────┘                      └──────────────────────┘
    │
    │ npm 解压 tarball → node_modules/react/
    │ npm 检查 package.json 中的依赖 → 尝试扁平化去重
    │ npm 生成 package-lock.json（精确版本记录）
    ▼
node_modules/ 中出现 react/ 目录
```

### 幽灵依赖与 node_modules 地狱

npm 的一个核心问题是**幽灵依赖**：你的 `package.json` 里没有声明某个包，但你的代码能 `import` 它——因为它被你的另一个依赖间接引入了。

```
你安装了 react（声明在 package.json 里）
react 依赖了 scheduler（你没声明）
但 scheduler 被扁平化提升到 node_modules/scheduler/

你的代码：import { unstable_scheduleCallback } from "scheduler";
→ 这行代码可以编译通过！
→ 但某天 react 更新 → 不再依赖 scheduler → 你的代码突然找不到这个包
```

`pnpm` 的解决方案：不像 npm 那样扁平化 node_modules 结构，pnpm 用**符号链接（symlink）** 建立一个严格的依赖图。每个包只能访问它明确声明的依赖——幽灵依赖现象消除。

### npm 注册中心的"不可变性"

npm 不会覆盖已发布的包。即使你 `npm unpublish`，同一版本号在 24 小时内不能再次发布——防止"恶意替换已发布的代码"。

这和 **crates.io**（Rust 的包注册中心）一样——Rust 社区不允许删除包。一旦发布就是永久的。这既是优点（安全审计），也是缺点（你没法"删掉发错的版本"）。

---

## Docker Hub / GHCR——容器镜像的存储和分发

### 你 `docker pull nginx` 时发生了什么

```
你的电脑
    │ docker pull nginx:1.27
    ▼
┌──────────────┐     HTTPS             ┌──────────────────────┐
│ Docker CLI   │ ──────────────────▶   │ registry-1.docker.io│
│ (本地工具)    │                       │ (Docker Hub 注册中心) │
│              │                       │                      │
│              │  ◀──────────────────   │ 镜像由多个"层"组成     │
│              │  1. 获取 manifest     │ nginx:1.27           │
│              │     (层列表+哈希)       │ ├── 层1: alpine:3.20 │
│              │  2. 并行下载每个层       │ ├── 层2: nginx binary│
│              │  3. 验证哈希           │ └── 层3: default.conf│
└──────────────┘                       └──────────────────────┘
    │
    │ Docker 解压各层 → 联合挂载 → 看到一个完整的文件系统 → 启动容器
    ▼
容器运行中
```

**为什么是"层"**：

```
alpine:3.20 层          被所有基于 Alpine 的镜像共享     100MB 只下载一次
nginx binary 层         Docker Hub 上只存一份            50MB
你的配置层               只有你自己的私有信息                2KB
```

Docker 的"层"就是基于 Content-Addressable Storage（CAS）。每个层由它的 SHA256 哈希唯一标识。如果两个镜像共享同一层（比如都基于 `alpine:3.20`），Docker 只下载它一次。

### GhostCache（GHCR）vs Docker Hub

- **Docker Hub**：官方注册中心。免费，但有拉取速率限制。匿名用户每 6 小时 100 次拉取
- **GHCR（GitHub Container Registry）**：GitHub 的容器注册中心。与 GitHub Actions 深度集成——你 CI 里构建的镜像直接推送到同账户下
- **ECR（AWS Elastic Container Registry）**：在 AWS 里用 K8s 时，从 ECR 拉取镜像比从 Docker Hub 快（因为是内网），而且没有速率限制

---

## GitHub Actions / GitLab CI——CI/CD 的运行机制

### 你 `git push` 后发生了什么

```
你的电脑                GitHub                        运行器（Runner）
    │                     │                                │
    │ git push            │                                │
    ├─────────────────────▶                                │
    │                     │ 检测到 .github/workflows/       │
    │                     │ deploy.yml 变更                │
    │                     │                                │
    │                     │ 分配一个运行器（VM）             │
    │                     ├───────────────────────────────▶│
    │                     │                                │ 1. 检出你的代码
    │                     │                                │ 2. 安装 Node.js 20
    │                     │                                │ 3. npm ci
    │                     │                                │ 4. npm run build
    │                     │                                │ 5. npm test
    │                     │                                │ 6. docker build
    │                     │                                │ 7. docker push
    │                     │                                │ 8. SSH 到服务器部署
    │                     │  ◀─────────────────────────────│
    │                     │  返回: 成功 ✓ / 失败 ✗           │
    │  ◀─────────────────┤                                │
    │  状态显示在你 PR      │                                │
    │  "所有检查通过"       │                                │
```

**这个 VM（运行器）存在多久**：几分钟到几小时（取决于你的 CI 时间）。**它是临时创建的，完成后销毁**。所以你不能在 CI 中"存东西"——要缓存依赖（node_modules），需要用 GitHub Actions 的 Cache Action 或上传到外部存储。

### GitLab CI 的区别

GitLab 的 CI 需要你部署自己的 **GitLab Runner**——一个在你自己服务器上运行的后台程序。当有人 push 时，GitLab 通知 Runner，Runner 拉取代码并执行流水线。部署方代码在同一个网络中（比如内网访问数据库），但你需要维护 Runner 的健康状态。

---

## Vercel / Netlify——不只是静态托管

### Vercel 在你 `git push` 后做了什么

```
开发者 git push

GitHub 通知 Vercel："这个仓库有新提交"

Vercel：
  1. 检测到 Next.js → 自动配置构建命令（next build）
  2. 分配一个容器 → 检出代码 → npm install → npm run build
  3. 构建产物：
     - 前端静态文件（.next/static/）→ 上传到 Vercel CDN（全球分发）
     - Serverless Functions（.next/server/）→ 部署到 Vercel 运行平台
     - Edge Middleware → 部署到 Vercel Edge Network
  4. 部署完成后 → 分配一个预览 URL（your-app-git-abc123.vercel.app）
  5. 如果是生产分支（main）→ 自动更新 your-app.vercel.app 的流量指向新版本

每个文件的缓存策略由 Vercel 自动管理：
  /static/js/main-abc123.js → 哈希名，不可变（永远缓存）
  / → 无哈希，可能更新（缓存时间短）
```

**Vercel 不是简单的 CDN + 文件服务器。** 它包含：

1. **Edge Network**：全球分布的 CDN 节点
2. **Serverless Functions**：你的 `src/app/api/*/route.ts` 被部署为 Serverless 函数
3. **Edge Middleware**：在 CDN 节点上运行的轻量 JS——可以在请求到达你的 API 之前做重定向、A/B 测试、认证检查
4. **ISR（Incremental Static Regeneration）**：你可以标记某个页面"缓存 60 秒，60 秒后下一个请求触发重新生成"——既有静态的速度，又有动态的更新
5. **自动 HTTPS**：Let's Encrypt 证书自动签 + 续

### Netlify 的区别

核心概念类似，但 Netlify 偏重"纯静态 + Serverless"，而不是 Next.js 深度集成。Netlify 还有表单处理（`<form netlify>`——不需要写 API 端点就能收表单提交）和身份认证（Netlify Identity——开箱即用的注册/登录/认证）。

---

## Cloudflare——互联网基础设施的隐形巨头

Cloudflare 不是竞争对手于 Vercel 或 AWS——它是**在它们前面**的一层。

### Cloudflare 的全球网络

```
用户 → Cloudflare CDN（距离最近的节点）
             │
             ├── 静态资源已在节点上 → 直接返回（最快）
             │
             ├── Worker（在节点上跑代码）
             │   → 认证检查 / A/B 测试 / URL 重写
             │
             └── 回源：请求最终到达你的服务器（Vercel / AWS / VPS）
                 → 动态 API
```

### Cloudflare 的多个产品，各是什么位置

| 产品 | 在架构中的位置 | 做什么 |
|------|-------------|--------|
| **DNS** | 域名解析 | 你在 Cloudflare 配 DNS，全球解析速度最快 |
| **CDN** | 静态文件分发 | 你的 JS/CSS/图片被缓存到 330+ 个全球节点 |
| **DDoS 防护** | 最外层保护 | 大规模恶意流量在到达你的服务器之前被过滤 |
| **Workers** | 边缘计算 | 在 CDN 节点上跑 JS 代码——不经过你服务器就能做很多事 |
| **Pages** | 静态托管 | 类似 Vercel/Netlify，纯静态 + Workers |
| **R2** | 对象存储 | S3 兼容，但**零出站带宽费用**——存储图片/视频成本极低 |
| **Tunnel** | 安全连接 | 从你的服务器到 Cloudflare 的安全隧道——不需要暴露公网 IP |
| **Zero Trust** | 访问控制 | 企业内部应用的认证网关 |
| **D1** | 边缘数据库 | SQLite 在 Cloudflare 节点上——Workers 可以直接访问，零延迟 |

### 为什么 Cloudflare Workers 是"范式转移"

传统后端：你的服务器在世界某处。用户从任何地方请求它——延迟受物理距离限制。London 用户访问你新加坡的服务器：~300ms 光速延迟，无法回避。

Worker：你的代码跑到每个 Cloudflare CDN 节点上。London 用户访问 London 的 Worker——延迟 ~5ms。

**但约束也很明显**：Worker 有 128MB 内存上限，CPU 时间 10-50ms，不能用 Node.js 原生模块（它是 V8 Isolate，不是 Node.js）。适合：认证检查、A/B 测试、URL 重写、小型计算。不适合：数据库查询、长时间处理。

---

## Let's Encrypt——为什么 HTTPS 是免费的

### HTTPS 证书的传统流程（2015 年之前）

```
1. 你生成一个 CSR（证书签名请求）
2. 你把它发给 CA（证书颁发机构，如 Verisign、Comodo）
3. CA 验证你确实拥有 example.com（人工邮件验证、电话验证）
4. CA 签发证书
5. 你把证书安装到 Nginx 上
6. 你付 $50-200/年
7. 1 年后证书过期，你重复步骤 1-6
```

### Let's Encrypt 的自动化（2016 年后）

```
1. Certbot（ACME 客户端）在你的服务器上运行
2. Certbot 生成密钥 + 向 Let's Encrypt 请求证书
3. Let's Encrypt 要求你证明你拥有这个域名：
   - 方法 1（HTTP Challenge）：Let's Encrypt 访问 http://example.com/.well-known/acme-challenge/xxx
     → 你的 Nginx 返回特定的 token → 证明你在控制这个域名的服务器
   - 方法 2（DNS Challenge）：你在 DNS 记录中放入特定 TXT 记录
     → 证明你可以修改这个域名的 DNS
4. 验证通过 → Let's Encrypt 签发证书 → Certbot 自动安装到 Nginx
5. Certbot 每月自动检查 → 证书过期前 30 天自动续期
6. 这一切都是免费的
```

Let's Encrypt 的所有费用由赞助商承担（Mozilla、Cisco、Google、Facebook、AWS 等），他们的动机是让整个互联网更安全（HTTPS 普及率从 2015 年的 40% 涨到 2024 年的 90%+）。2015 年前，许多小型网站用 HTTP 因为证书太贵太麻烦——Let's Encrypt 彻底消除了这两个问题。

---

## 下一代包管理器——npm 时代之后

npm 是 2010 年存在的解决方案。随着生态的发展，新的问题出现了：

| 问题 | npm 的局限 | 新的包管理器怎么解决 |
|------|-----------|-------------------|
| 安装速度慢 | 顺序下载、不共享依赖 | pnpm——符号链接去重；bun——全用二进制格式 |
| 幽灵依赖 | node_modules 扁平化 | pnpm——严格的引用结构，只能访问声明过的依赖 |
| 全局工具污染 | `npm install -g` 影响所有项目 | npx——临时运行包；corepack——声明项目级别的包管理器版本 |
| npm 注册中心的中央化 | 只有一个 registry | jsr——新包注册中心（Deno 主导），兼容 TypeScript 原生 |

**JSR（JavaScript Registry）** 是 2024 年后值得关注的新注册中心——它为 TypeScript 设计，不兼容 CommonJS。目标不是取代 npm，而是提供一个现代的、类型安全的、跨运行时的（Deno + Node.js + Bun）的替代方案。

---

## 文件后缀参考（附录）

见 [文件后缀速查](appendix/file-extensions.md)。

---

## 架构目录到此结束

你已经知道：
1. [运行时模型](01-runtime-model.md)——代码跑在哪、怎么通信
2. [构建与编译](02-build-and-compile.md)——源码怎么变成可运行的产物
3. [项目组织](03-project-organization.md)——前后端代码的物理关系
4. [部署全景](04-deployment.md)——从构建产物到可访问的网站
5. [技术选型](05-tech-selection.md)——在每个生态位上怎么选择
6. [生产场景模式](06-production-patterns.md)——真实项目的技术栈组合
7. [生态基础设施](07-ecosystem-infra.md)——那些"隐形"的基础设施在做什么

下一步：带着这个框架，回到 [domains/](../domains/) 或 [languages/](../languages/)，你能准确判断每项技术在整体架构中的位置。
