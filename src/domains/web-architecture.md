# Web 架构全景：从代码到用户屏幕的完整物理路径

本文回答一个问题：**你写的代码到底在哪里跑？怎么从编辑器里的文件变成用户在浏览器里看到的东西？**

这不是技术选型指南，不是代码教程。这是**物理现实**——每个文件跑在哪台机器上、谁编译它、谁传输它、谁接收它。

读完本文后：
- 你知道 HTML/CSS/JS 各自负责什么、浏览器怎么把它们合成一个页面
- 你知道"前端代码调后端 API"这九个字背后发生了多少步
- 你知道一台物理服务器、一个 Web 服务器进程、一个公网 IP、一个端口之间的精确关系
- 你知道 nginx 是干什么的、为什么几乎所有生产环境都有它
- 你知道"部署"到底做了什么——编译产物去了哪里

---

## 一、物理现实：代码跑在哪里

### 1.1 前端代码在哪里运行

**前端代码运行在用户的浏览器里。**

你写的每个 `.ts`、`.tsx`、`.js` 文件，经过编译后变成 JavaScript 文件，被浏览器下载到**用户的电脑上**，由浏览器的 JavaScript 引擎（Chrome 的 V8、Firefox 的 SpiderMonkey、Safari 的 JavaScriptCore）执行。

```
你的电脑（开发机）                         用户的电脑
┌──────────────────┐                    ┌──────────────────┐
│ 你写的 page.tsx   │  ──编译──▶         │  浏览器           │
│ 你写的 Button.tsx │  ──编译──▶         │  ├─ JS 引擎（V8）  │
│ 你写的 style.css  │  ──复制──▶         │  ├─ 渲染引擎       │
│                   │                    │  └─ 网络栈         │
└──────────────────┘                    └──────────────────┘
```

**关键含义**：
- 前端代码不在你的服务器上运行——在访问者的电脑上运行
- 任何人都可以在浏览器开发者工具里看到你的前端源码（即使经过了压缩和混淆）
- 前端代码不能直接访问数据库、不能直接读写服务器文件——如果它能，任何一个用户都能操作你的数据库

### 1.2 后端代码在哪里运行

**后端代码运行在一台持续开机、联网的电脑上**。这台电脑通常是一台租来的服务器（物理机或虚拟机），不是你桌上的开发机。

```
                              服务器（租来的，在某个数据中心）
用户电脑                        ┌─────────────────────────┐
┌──────────┐    HTTP 请求      │  后端进程（你写的代码）    │
│ 浏览器    │ ──────────────▶  │  ├─ 监听 8080 端口       │
│          │ ◀──────────────  │  ├─ 处理请求              │
│          │    HTTP 响应      │  ├─ 查询数据库            │
└──────────┘                   │  └─ 返回 JSON            │
                               └─────────────────────────┘
```

**关键理解**：

**物理服务器** ≠ **Web 服务器**

"物理服务器"是一台**真实的机器**（或虚拟机）——有机箱、CPU、内存、硬盘、网卡。"Web 服务器"有两种含义：

| 含义 | 是什么 | 例子 |
|------|--------|------|
| **Web 服务器软件**（进程） | 在物理服务器上运行的一个程序，监听某个端口，等待 HTTP 请求 | Nginx、Apache、你写的 Go/Python/Node.js 程序 |
| **Web 服务器硬件** | 专门用于运行 Web 服务程序的物理机器 | 阿里云 ECS、AWS EC2 实例 |

**一台物理服务器上可以同时运行多个 Web 服务器进程**：

```
一台物理服务器（IP: 203.0.113.5）
├── Nginx（进程，监听 80 和 443 端口）
├── Node.js API（进程，监听 3001 端口）
├── Go 服务（进程，监听 3002 端口）
├── Python 后台任务（进程，不监听端口，定时执行）
└── PostgreSQL（进程，监听 5432 端口）
```

每个进程独立运行。进程之间通过操作系统的进程隔离（独立的内存空间）互不影响。如果 Node.js 进程崩溃，Go 服务不受影响。

### 1.3 浏览器也是一个进程

你的浏览器在你电脑上也是一个普通的进程。它的特殊之处在于它包含了一个**JavaScript 引擎**（执行 JS 代码）和一个**渲染引擎**（把 HTML/CSS 画成像素）。

```
你的电脑
┌─────────────────────────────────────────────────────┐
│  浏览器                                             │
│  ├── 网络栈：发送 HTTP 请求，下载 HTML/CSS/JS/图片    │
│  ├── JS 引擎（V8）：执行下载的 JavaScript 代码        │
│  ├── 渲染引擎：解析 HTML/CSS，画成像素，显示在屏幕上   │
│  └── 存储：Cookie、localStorage、IndexedDB           │
└─────────────────────────────────────────────────────┘
```

---

## 二、前端三件套：HTML、CSS、JS 是什么关系

**这三者不是三种"编程语言"——它们在浏览器里有明确的分工。**

### 2.1 各自负责什么

| | HTML | CSS | JavaScript |
|------|------|------|-------------|
| **角色** | 骨架——结构 | 皮肤——外观 | 肌肉——行为 |
| **做什么** | 定义页面上有哪些元素（按钮、文字、图片、输入框） | 定义这些元素长什么样（颜色、大小、位置、间距） | 定义元素的行为（点击按钮后发生什么、数据从哪来） |
| **类比** | 建筑的结构图 | 建筑的装修方案 | 建筑的电路和水管——让东西动起来 |
| **浏览器怎么看** | 解析成 DOM 树 | 解析成 CSSOM 树，挂到 DOM 上 | 执行后可以修改 DOM 和 CSSOM |

### 2.2 它们不是"分别完成不同部分然后拼接"

更准确的表述是：**HTML 提供结构，CSS 提供外观规则，JS 在运行时动态修改结构和外观。**

```
浏览器加载一个页面的过程：

第 1 步：下载 HTML 文件
        <html>
          <body>
            <button id="btn">点击</button>
          </body>
        </html>

第 2 步：解析 HTML → 构建 DOM 树
        document
          └── html
              └── body
                  └── button#btn (文本: "点击")

第 3 步：遇到 <link> 标签 → 下载 CSS 文件 → 构建 CSSOM 树
        button { color: white; background: blue; }

第 4 步：DOM 树 + CSSOM 树 → 合成为渲染树
        button#btn → 颜色: 白色, 背景: 蓝色, 显示位置: (x, y), 大小: (w, h)

第 5 步：布局（Layout）——计算每个元素的确切位置和大小
        button#btn → x=100, y=200, w=80, h=40

第 6 步：绘制（Paint）——把每个像素的颜色写入显存
        [白][白][白][蓝][蓝][蓝][蓝][蓝][白][白] ...

第 7 步：遇到 <script> 标签 → 下载 JS 文件 → V8 引擎执行
        const btn = document.getElementById("btn");
        btn.addEventListener("click", () => { ... });

        这段 JS 找到 button 对象，给它挂了一个点击回调。
        用户点击时 → JS 修改 DOM（比如 button.textContent = "已点击"）
                   → 浏览器重新执行第 4-6 步（重新渲染）
```

**关键理解**：

- HTML 和 CSS 是**声明式的**——你写"这里有个按钮，它是蓝色的"，浏览器负责实现
- JS 是**命令式的**——你写"找到这个按钮，当用户点击它时，把它的文字改成 X"
- 这三者不是"三个功能模块并行工作"——是**浏览器先解析 HTML/CSS 建好框架，JS 在这个框架上做动态修改**

### 2.3 JS 为什么能改 HTML 和 CSS

因为浏览器把 HTML 和 CSS **暴露为 JS 可以操作的对象**：

```
HTML 元素 → 浏览器解析为 → DOM 节点 → JS 通过 document.getElementById() 拿到 → 可以改

CSS 样式 → 浏览器解析为 → CSSOM 样式规则 → JS 通过 element.style.color 拿到 → 可以改
```

**浏览器运行时（Browser Runtime）就是这个桥梁**。它把 HTML/CSS 的静态描述转换为 JS 可以操作的动态对象。

---

## 三、前后端如何通信

### 3.1 HTTP 请求的本质

前端和后端之间通过 **HTTP 协议**通信。HTTP 是建立在 TCP 之上的文本协议：

```
前端代码（浏览器里跑的 JS）：
  fetch("https://api.example.com/users/alice")

  浏览器把这个函数调用翻译成：
  ┌──────────────────────────────────────────────────┐
  │  GET /users/alice HTTP/1.1                      │
  │  Host: api.example.com                          │
  │  Authorization: Bearer eyJhbGciOi...             │
  │  Accept: application/json                       │
  │                                                 │
  └──────────────────────────────────────────────────┘
          │
          ▼  通过互联网到达服务器
          │
  后端代码（服务器上跑的进程）收到这段文本
  → 解析：方法是 GET，路径是 /users/alice
  → 调用对应的处理函数
  → 返回：
  ┌──────────────────────────────────────────────────┐
  │  HTTP/1.1 200 OK                                │
  │  Content-Type: application/json                 │
  │                                                 │
  │  {"id": 1, "name": "Alice", "email": "..."}     │
  └──────────────────────────────────────────────────┘
          │
          ▼  通过互联网返回浏览器
          │
  浏览器收到 → JS 拿到 JSON 对象 → 更新界面
```

### 3.2 前端代码怎么"知道"后端在哪

前端代码里写 API 调用时，需要一个 URL。这个 URL 怎么来？

**方案 A：写在代码里（开发时）**

```typescript
// 前端代码里硬编码
const response = await fetch("http://localhost:3001/api/users");
// "localhost:3001" 是"本机的 3001 端口"
// 开发时后端在本机跑，所以可以访问
```

**方案 B：通过环境变量（生产时）**

```typescript
// 前端代码里不写具体域名，用变量代替
const response = await fetch(`${process.env.NEXT_PUBLIC_API_URL}/api/users`);
// 构建时，这个变量被替换为实际的值
// 开发环境：http://localhost:3001
// 生产环境：https://api.example.com
```

**方案 C：前后端同域（Next.js 的常见模式）**

如果前端和后端在同一个域名下（比如都在 `example.com`），请求可以写相对路径：

```typescript
const response = await fetch("/api/users");
// 浏览器自动补全为：https://example.com/api/users
// 因为当前页面的域名是 example.com
```

### 3.3 "调用 API"到底是什么意思

API（Application Programming Interface）不是一种特殊的技术——**就是一段路径的约定**。

```
后端写了一堆"处理函数"，每个函数对应一个 URL 路径：

GET    /api/users           →  返回所有用户
GET    /api/users/:id       →  返回 ID 对应的用户
POST   /api/users           →  创建一个新用户
PUT    /api/users/:id       →  更新某个用户
DELETE /api/users/:id       →  删除某个用户

这就叫 API。本质上就是：
"如果你用 GET 方法访问 /api/users/42，我就返回 ID 为 42 的用户信息"
```

前端代码不需要知道后端是用 Python、Go 还是 Node.js 写的——只需要知道"这个 URL 返回什么格式的数据"。**HTTP 协议隔离了前端的语言和后端的语言。**

---

## 四、前后端是同一个项目吗

### 4.1 三种常见方式

```
方案 A：前后端分仓库（传统）
  frontend/    ← 一个 Git 仓库，Next.js 项目，部署到 Vercel
  backend/     ← 另一个 Git 仓库，Go 项目，部署到 AWS
  优点：团队可以独立开发、独立部署、独立扩缩容
  缺点：API 变更需要前后端同步，没有"单次提交改全链路"

方案 B：Monorepo（前后端同仓库，分开部署）
  my-project/
  ├── apps/
  │   ├── frontend/   ← Next.js 项目
  │   └── backend/    ← Go 项目
  ├── packages/       ← 共享的类型定义和工具库
  └── package.json
  优点：一个 PR 可以同时改前后端，共享类型定义保证 API 一致性
  缺点：仓库体积大，构建配置复杂

方案 C：前后端同项目、同部署（Next.js 全栈）
  my-app/
  ├── src/
  │   ├── app/
  │   │   ├── page.tsx        ← 前端：首页组件
  │   │   └── api/
  │   │       └── users/
  │   │           └── route.ts  ← 后端：API 处理函数
  │   └── lib/
  │       └── db.ts           ← 数据库连接（前后端都可能用到）
  └── package.json            ← 前后端共享一个 package.json
  同一个 Next.js 项目，既提供前端页面，又提供后端 API。
  Vercel 部署时，前后端自动部署到一起。
```

### 4.2 前端编译 ≠ 后端编译

**前端和后端的编译过程完全不同**：

```
前端编译（Vite / Next.js 构建）：
  src/app/page.tsx           ──▶  .next/static/chunks/app-page-abc123.js
  src/components/Button.tsx  ──▶  .next/static/chunks/component-button-def456.js
  src/styles/globals.css     ──▶  .next/static/css/globals-hash789.css

  TS/JSX/CSS → JS bundle + CSS bundle → 静态文件
  放在服务器的 .next/ 目录（或导出为 out/ 目录的纯静态文件）

后端编译：
  src/app/api/users/route.ts ──▶  .next/server/app/api/users/route.js
  （看起来还是 JS，但运行环境不同——Node.js 服务器端，不是浏览器）

  TS → JS（Node.js 可执行）
  放在服务器的 .next/server/ 目录
```

**关键区别**：

| | 前端代码编译后 | 后端代码编译后 |
|------|-------------|-------------|
| **跑在哪** | 用户的浏览器 | 服务器的 Node.js 进程 |
| **产物** | JS bundle（单文件或少量文件） | JS 文件（目录结构保持） |
| **可以访问什么** | 浏览器 API（DOM、localStorage、fetch） | Node.js API（文件系统、数据库驱动、环境变量） |
| **谁能看到源码** | 任何访问者（压缩后的 JS 在浏览器中可见） | 只有服务器管理员 |

### 4.3 编译产物放在哪里

```
开发时（npm run dev）：
  编译产物在内存中——Vite/Next.js 的开发服务器不写磁盘
  每次请求动态编译，所以你改文件立即生效

构建时（npm run build）：
  前端产物 → .next/static/（或 out/）——静态 JS/CSS/HTML 文件
  后端产物 → .next/server/（或 dist/）——Node.js 可执行的 JS 文件

部署时（npm start 或部署到 Vercel/服务器）：
  把整个 .next/ 目录（含前端静态文件 + 后端代码）复制到服务器
  服务器上运行 node server.js 或 next start
```

---

## 五、IP、端口、DNS、Nginx

### 5.1 物理服务器和公网 IP 的关系

**一台物理服务器有一个或多个公网 IP 地址**，不是每个 Web 服务一个 IP。

```
物理服务器（在阿里云/ AWS 租的）
├── 公网 IP: 203.0.113.5（全世界通过这个 IP 访问这台服务器）
│
├── Nginx（进程，监听 80 端口）    ← 来自互联网的请求首先到达这里
├── Node.js API（进程，监听 3001） ← 不对外暴露，只有 Nginx 转发过来的请求
├── Go 服务（进程，监听 3002）      ← 同上
└── PostgreSQL（进程，监听 5432）   ← 只监听 127.0.0.1:5432，外部无法直接访问
```

**同一台物理服务器上可以有 N 个 Web 服务，它们通过端口号区分。外部只能通过 Nginx（80/443）进入，Nginx 根据规则转发到内部端口。**

### 5.2 公网 IP 是怎么获得的

```
你租了一台云服务器（阿里云 ECS / AWS EC2 / Vultr VPS）
  → 云厂商从它们的 IP 池中分配一个公网 IP 给你的实例
  → 这个 IP 是全球唯一的，你的服务器独享

你买了一台物理服务器，放在数据中心
  → 数据中心从它们向 ISP 购买的 IP 段中分配一个 IP 给你
  → 你每月为这个 IP 和带宽付费
```

没有免费的公网 IP。每个公网 IP 都是某个 ISP（互联网服务提供商）的资产，层层分配下来的。

### 5.3 DNS：域名到 IP 的映射

用户输入 `example.com`，浏览器不知道这个域名对应哪个 IP。DNS 负责翻译：

```
浏览器输入 example.com
    │
    ▼
查询本地 DNS 缓存（浏览器缓存 → OS 缓存）
    │ 没找到
    ▼
查询 ISP 的 DNS 服务器（如 8.8.8.8 / 1.1.1.1）
    │ 没找到
    ▼
查询根 DNS → 找到 .com 的权威 DNS
    │
    ▼
查询 .com 的权威 DNS → 找到 example.com 的权威 DNS
    │
    ▼
example.com 的权威 DNS 返回：203.0.113.5
    │
    ▼
浏览器拿到 IP 地址，向 203.0.113.5:443 发起 HTTPS 连接
```

**DNS 配置**：你在域名注册商（Namecheap、Cloudflare、阿里云）那里设置了一条 A 记录：

```
example.com  →  A  →  203.0.113.5
```

意思是"把 example.com 解析到这个 IP"。全世界任何地方解析这个域名都会得到同一个 IP。

### 5.4 Nginx 是做什么的

**Nginx 运行在你的服务器上，监听 80（HTTP）和 443（HTTPS）端口，是所有外部请求的统一入口。**

```
互联网请求 → 你的服务器 203.0.113.5
                │
                ▼
            Nginx（监听 80 + 443）
                │
                │ 根据请求的域名和路径，决定转发给谁：
                │
                ├── api.example.com → 转发到 localhost:3001（Node.js API）
                ├── app.example.com → 返回 /var/www/app/index.html（静态前端文件）
                ├── admin.example.com → 转发到 localhost:3002（Go 管理后台）
                └── /images/* → 直接从 /var/www/images/ 返回静态文件
```

**为什么需要 Nginx？**

1. **单一入口**：公网只有一个 80/443 端口。没有 Nginx，多个后端服务没法共享这两个端口
2. **HTTPS 终结**：SSL 证书配置一次在 Nginx 上，所有后端服务自动获得 HTTPS（后端服务之间走 HTTP，简单且快）
3. **静态文件服务**：前端的编译产物（JS/CSS/HTML）不需要一个 Node.js 进程来返回——Nginx 直接从磁盘读取并返回，速度极快
4. **负载均衡**：如果流量大，一个后端服务不够——Nginx 可以把请求均匀分发给多个后端实例
5. **安全隔离**：后端服务只监听 `127.0.0.1:3001`（仅本机可访问），外部无法直接攻击后端

```
普通的 Nginx 配置：
                    监听 80 端口（HTTP），自动重定向到 443（HTTPS）
server {
    listen 80;
    server_name example.com;
    return 301 https://$host$request_uri;
}

                    监听 443 端口（HTTPS），根据路径分发
server {
    listen 443 ssl;
    server_name example.com;        ← 只处理 example.com 的请求

    ssl_certificate /etc/ssl/cert.pem;
    ssl_certificate_key /etc/ssl/key.pem;

                    访问 example.com/ 或 /about 时：
                    返回前端构建产物（纯静态文件，Nginx 直接从磁盘读）
    location / {
        root /var/www/frontend;
        try_files $uri /index.html;
    }

                    访问 example.com/api/ 开头时：
                    转发给本机的 Node.js 进程（3001 端口）
    location /api/ {
        proxy_pass http://localhost:3001;
        proxy_set_header X-Forwarded-For $remote_addr;
    }
}
```

---

## 六、完整请求旅程：从 URL 到页面显示

把上面所有概念串起来，看一个完整的请求：

### 步骤 1：用户输入 URL

```
用户在新标签页输入 https://example.com → 按回车
```

### 步骤 2：浏览器查找 IP

```
浏览器：example.com 对应哪个 IP？
  → 查 DNS 缓存：没找到
  → 查 ISP 的 DNS：返回 203.0.113.5
```

### 步骤 3：建立 TCP 连接，发 HTTPS 请求

```
浏览器向 203.0.113.5:443 发起 TCP 连接
  → 三次握手（SYN → SYN-ACK → ACK）
  → TLS 握手（交换密钥、验证证书）
  → 在加密通道里发送：

GET / HTTP/1.1
Host: example.com
Accept: text/html, application/json
```

### 步骤 4：Nginx 接收请求并转发

```
请求到达你的服务器 203.0.113.5:443
  → Nginx 收到：请求的是 example.com，路径是 /
  → Nginx 检查自己的配置：
      - 这个 server_name 是 example.com
      - / 路径 → 返回静态文件
  → Nginx 从 /var/www/frontend/index.html 读取文件
  → 返回给浏览器
```

### 步骤 5：浏览器收到 HTML

```html
<!DOCTYPE html>
<html>
<head>
  <link rel="stylesheet" href="/static/css/globals.css">
</head>
<body>
  <div id="root"></div>
  <script src="/static/js/main.js"></script>
</body>
</html>
```

### 步骤 6：浏览器递归下载依赖资源

```
解析 HTML → 发现 <link href="/static/css/globals.css"> → 下载 CSS 文件
          → 发现 <script src="/static/js/main.js"> → 下载 JS 文件

每次下载都经历步骤 2-4（DNS 已缓存，直接从 TCP 连接开始）
```

### 步骤 7：浏览器渲染页面

```
HTML 解析 → DOM 树
CSS  解析 → CSSOM 树
DOM + CSSOM → 渲染树 → 布局（计算每个元素的坐标和大小） → 绘制（填充像素）

此时用户看到了页面——但可能只是一个骨架（空 div），数据还没加载
```

### 步骤 8：JS 执行，发起 API 请求

```
main.js 执行 → React 组件渲染 → useEffect 触发 → fetch("/api/users") 调用

    GET /api/users HTTP/1.1
    Host: example.com
    Authorization: Bearer <token>
```

### 步骤 9：Nginx 转发 API 请求

```
Nginx 收到：请求的是 example.com/api/users
  → 匹配到 location /api/ 规则
  → 转发给 http://localhost:3001/api/users
```

### 步骤 10：后端处理请求

```
Node.js 进程（监听 3001 端口）收到：
  GET /api/users
  Authorization: Bearer <token>

  → 验证 token（确认这个用户已登录）
  → 查询数据库：SELECT * FROM users WHERE org_id = $1
  → 组织响应：JSON 数组
  → 返回 HTTP 200 + JSON
```

### 步骤 11：前端收到数据，更新页面

```
浏览器收到 JSON：[{id:1, name:"Alice"}, {id:2, name:"Bob"}]
  → React 的 setUsers(data) 被调用
  → 状态更新 → 组件重新渲染 → 新 JSX → diff → 更新 DOM
  → 用户看到用户列表出现在页面上
```

### 步骤 12：页面完全加载

用户看到的是完整的、可交互的页面。整个过程通常在 1-3 秒内完成。

---

## 七、最常见的困惑解答

### "前后端代码是一个项目吗？"

可以是一个项目（Next.js 方案 C），也可以是两个独立项目（方案 A）。但**即使是一个项目，前端代码跑在浏览器、后端代码跑在服务器，它们的运行环境完全不同。**

### "后端编译好后放在哪里？"

如果后端是 Node.js：编译后的 JS 文件放在服务器的文件系统上（如 `/opt/app/dist/`），由一个进程管理器（如 systemd 或 PM2）启动 Node.js 进程来执行。

如果后端是 Go：编译成一个静态二进制文件（如 `/opt/app/server`），直接执行这个文件。

### "前端编译好后放在哪里？"

前端编译产物是纯静态文件（HTML + JS + CSS + 图片）。放在服务器上任意的目录（如 `/var/www/frontend/`），由 Nginx 直接返回给浏览器。也可以放在专门的静态托管服务上（如 Vercel、Netlify、Cloudflare Pages）。

### "我自己的电脑上怎么跑前后端？"

```
开发模式下：

前端（npm run dev）：
  Vite 在 localhost:5173 启动开发服务器
  你打开浏览器访问 localhost:5173 → 看到页面

后端（npm run dev 或 go run .）：
  Node.js 在 localhost:3001 启动后端服务器
  前端代码里写的是 fetch("http://localhost:3001/api/users")

生产模式下（模拟）：
  前端构建：npm run build → .next/static/
  后端构建：npm run build → .next/server/
  启动：npm run start → Next.js 同时提供前端静态文件和后端 API
  你打开浏览器访问 localhost:3000 → 一切都由这一个端口提供
```

### "为什么需要 Nginx？Node.js 自己也能监听 80 端口"

Node.js 可以直接监听 80 端口，生产环境中也有这样用的。但 Nginx 提供了 Node.js 不擅长的事情：

- 高效的静态文件服务（从磁盘一次性读到内存并缓存，之后零开销）
- HTTPS 证书管理（Let's Encrypt 自动续期）
- 限流和防盗链
- 一个域名服务多个应用
- 作为"保护层"——Node.js 进程只暴露给本机，减少攻击面

在服务器生态中，**让专门的工具做专门的事**是核心原则。Nginx 擅长处理连接、返回静态文件；Node.js 擅长业务逻辑。各司其职。

### "公网 IP 和端口——到底怎么访问"

```
我租了一台 AWS 服务器，公网 IP 是 54.12.34.56

我在这台服务器上：
1. 安装了 Nginx，监听 80 和 443
2. 装了 Node.js API，监听 localhost:3001
3. 配了 Nginx 把 /api/* 转发到 localhost:3001

你（用户）在浏览器输入：https://54.12.34.56/api/users
  → 浏览器向 54.12.34.56:443 发 HTTPS 请求
  → Nginx 在 443 端口收到请求
  → 看到路径是 /api/users → 转发给 localhost:3001
  → Node.js 在 3001 端口收到（Nginx 转发过来的）
  → 处理 → 返回 JSON → Nginx 转给浏览器

如果你在浏览器输入：https://54.12.34.56:3001/api/users
  → 浏览器向 54.12.34.56:3001 发请求
  → 但 Node.js 只监听 localhost:3001（127.0.0.1:3001），不监听公网 IP 的 3001 端口
  → 连接被拒绝

     这就是"只暴露给本机"的含义——
  localhost:3001 = 只有这台服务器自己可以访问 3001 端口
  公网 IP:3001  = 全世界的任何机器都可以访问 3001 端口（如果你监听了 0.0.0.0:3001）
```

---

## 八、与现有文档的关系

| 你想知道什么 | 读哪个文档 |
|------------|-----------|
| 客户端-服务端模式在 OS 每一层的普遍性 | [系统架构](system-architecture.md) |
| HTML/CSS/JS/框架/元框架/组件库各是干嘛的、怎么协同 | [前端开发全景](frontend.md) |
| TCP → HTTP → 并发 → 数据库 → 缓存 → 消息队列 → 部署 的演进 | [后端开发全景](backend.md) |
| 每门语言在这些维度上做了什么选择 | [topics/](../topics/) 系列文档 |
| 当前各语言和框架的流行度、学哪个值 | [生态现状](ecosystem-state.md) |
| 用什么参考项目学习全栈 | [参考项目对比](reference-projects.md) |
| 所有开发领域（Web/移动/桌面/游戏/嵌入式）的概念全貌 | [开发生态大地图](development-ecosystem.md) |

**本文是其他所有文档的"物理基础"——在讨论任何具体技术之前，先搞清楚代码跑在哪、怎么连接、数据怎么流。**
