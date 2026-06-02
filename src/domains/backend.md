# 后端开发全景指南

> **前置阅读**：后端涉及多语言选择——建议先阅读对应语言的全景指南（[Go](../languages/go.md)、[Python](../languages/python.md)、[Rust](../languages/rust.md)、[JavaScript](../languages/javascript.md)）。

---

## 起点：什么是后端

后端是运行在**服务器**上的程序。它的工作可以用一句话描述：

```
接收请求 → 处理逻辑 → 返回响应
```

一台机器上的一个程序，打开一个端口，等待网络连接。当连接到来时，读取数据（请求），根据数据做计算或查询，返回结果（响应）。

```python
# 这六行就是一个完整的后端程序
from http.server import HTTPServer, BaseHTTPRequestHandler
class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        self.end_headers()
        self.wfile.write(b"hello")
HTTPServer(("0.0.0.0", 8080), Handler).serve_forever()
```

所有后端技术的复杂度，都是从"六行代码能跑"到"服务百万用户"的过程中一层一层加上去的。本文按照这个演进顺序来组织。

---

## 第一层：从 TCP 到 HTTP

### 网络通信的基础

两台计算机通过网络通信，底层是 TCP/IP 协议。TCP 提供了可靠的双向字节流：A 发给 B 的数据，保证按顺序、不丢失地到达。

但 TCP 只负责传输**字节流**。浏览器和服务器需要约定这些字节的含义——这就是 **HTTP 协议**。

### HTTP 的结构

```
客户端发送（请求）：                    服务器返回（响应）：
GET /users/alice HTTP/1.1             HTTP/1.1 200 OK
Host: example.com                     Content-Type: application/json
Accept: application/json              Content-Length: 45
                                      {"id": 1, "name": "Alice", "email": "..."}
```

| 部分 | 请求 | 响应 |
|------|------|------|
| **起始行** | 方法（GET/POST/PUT/DELETE）+ 路径 + HTTP 版本 | HTTP 版本 + 状态码（200/404/500） |
| **头部（Headers）** | Host、Accept、Authorization | Content-Type、Content-Length、Cache-Control |
| **主体（Body）** | POST/PUT 时附带的数据（JSON、表单） | 返回的数据（HTML、JSON、二进制） |

### 状态码的含义

| 范围 | 含义 | 常见 |
|------|------|------|
| 1xx | 信息 | — |
| 2xx | 成功 | 200 OK, 201 Created, 204 No Content |
| 3xx | 重定向 | 301 Moved Permanently, 302 Found, 304 Not Modified |
| 4xx | 客户端错误 | 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 429 Too Many Requests |
| 5xx | 服务器错误 | 500 Internal Server Error, 502 Bad Gateway, 503 Service Unavailable |

---

## 第二层：从单请求到并发——服务器如何同时服务多个用户

### 问题

如果一个请求需要 1 秒来处理（比如查询数据库），而你只有一个处理线程，那么在这 1 秒内到达的其他 99 个用户都必须在队列中等待。第 100 个用户要等 100 秒。

**这就是并发问题的根源**：单个请求处理不是瞬间完成的，而用户同时到达。

### 解决方案的演进

**方案 A：多进程（1990s，Apache prefork）**

```
请求到达 → 主进程 fork 一个子进程处理这个请求 → 处理完子进程销毁
```

每个请求一个独立进程。进程之间完全隔离（一个崩溃不影响其他）。但每个进程占用独立内存（~10MB），1000 个并发请求 = 10GB 内存。

**方案 B：多线程（Apache worker、早期 Java Servlet）**

```
请求到达 → 从线程池中拿一个空闲线程处理 → 处理完归还线程
```

线程比进程轻量（共享内存空间），但仍然受限于操作系统——几百到几千个线程是上限。而且共享内存带来了数据竞争风险。

**方案 C：事件循环 + 非阻塞 I/O（Node.js, 2009—）**

```
一个主线程，一个循环。
有新请求到来 → 注册一个回调 → 继续处理其他事情。
当数据库查询完成 → 触发回调 → 处理结果 → 返回响应。
```

核心思想：**不要在等待 I/O 的时候阻塞线程**。80% 以上的请求处理时间花在等待（数据库查询、外部 API 调用、文件读写）。事件循环模型在等待时不占用线程，可以同时处理数万个并发连接。

代价：代码变成了回调嵌套（callback hell）。Promise 和 async/await 是后来加上的补救。

**方案 4：轻量级协程——goroutine（Go，2009—）**

```
每个请求一个 goroutine（~2KB 栈，可同时运行百万个）。
Go 运行时自动把 goroutine 多路复用到少数 OS 线程上。
代码写成同步风格（顺序执行），底层是异步 I/O。
```

Go 的答案是：让开发者用最简单的"同步代码"风格写并发，运行时负责所有的异步调度。这是目前在"开发体验"和"并发性能"之间最好的折中之一。

**方案 5：Virtual Threads（Java 21，2023—）**

Java 的最新答案。与 goroutine 理念相同——轻量级用户态线程，同步代码风格。

### 并发模型的权衡总结

| 模型 | 语言 | 并发上限 | 代码复杂度 | 内存效率 |
|------|------|---------|----------|---------|
| 多线程 | Java（传统）、C++，Python（GIL 限制） | 数百-数千 | 中 | 低 |
| 事件循环 | Node.js、Python asyncio | 数万 | 高（async/await 渗透全栈） | 高 |
| 轻量协程 | Go、Java 21+ | 数十万-百万 | 低（同步代码风格） | 高 |

---

## 第三层：数据持久化——数据库的出现

### 问题

如果服务器进程重启，所有存储在内存中的数据都丢失了。而且当你有多台服务器时，它们无法共享内存。

**你需要一个独立于服务器进程的数据存储。**

### 关系型数据库

关系型数据库的核心组织方式是**表**——类似 Excel 工作表。每行是一条记录，每列是一个字段。

```sql
-- 定义结构
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT UNIQUE NOT NULL
);

-- 写入
INSERT INTO users (name, email) VALUES ('Alice', 'alice@example.com');

-- 查询
SELECT name, email FROM users WHERE id = 1;
```

**为什么大多数应用首选关系型数据库？**

1. **Schema**（模式）：在写入数据前就定义了数据的结构和约束，防止垃圾数据进入
2. **JOIN**：关联查询。从"用户"表和"订单"表中同时查询"姓 Alice 的所有订单"
3. **事务（Transaction）**：一组操作要么全部成功，要么全部回滚。转账时不会出现"扣了钱但没加上去"
4. **ACID**：原子性、一致性、隔离性、持久性——关系型数据库最核心的保证

**主要选择**：

| 数据库 | 特点 |
|--------|------|
| **PostgreSQL** | 功能最全、扩展性最强。支持 JSON、全文搜索、地理空间。**新项目的默认选择** |
| **MySQL / MariaDB** | 被广泛使用（WordPress、大量老项目）。复制方案成熟 |
| **SQLite** | 嵌入式（不是客户端-服务器架构）。单文件、零配置。移动端、小型项目、Edge 部署的首选 |

### NoSQL 数据库

并非所有数据都适合放在规整的表格里。

| 类型 | 代表 | 使用场景 |
|------|------|---------|
| **键值存储** | Redis | 缓存、会话存储、限流计数器、排行榜 |
| **文档数据库** | MongoDB | Schema 不固定、快速迭代、嵌套结构 |
| **宽列存储** | Cassandra、ScyllaDB | 海量时序数据（物联网、日志） |
| **搜索引擎** | Elasticsearch、Meilisearch | 全文搜索 |
| **图数据库** | Neo4j | 社交关系、推荐系统 |

**关键判断**：大多数应用用 PostgreSQL + Redis 就够了。NoSQL 不是在"替代" SQL，而是不同场景的不同工具。

### 应用和数据库之间的通信

**直接写 SQL**：
```python
cursor.execute("SELECT name, email FROM users WHERE id = %s", [user_id])
```
精确控制，但容易写出 SQL 注入漏洞（永远用参数化查询！）、重复性代码多、重构字段名麻烦。

**ORM（Object-Relational Mapping）**：
```python
# 操作 Python 对象，ORM 自动生成 SQL
user = User.objects.get(id=user_id)
```
不用写 SQL。但复杂查询难以表达、可能生成低效的 SQL（N+1 查询）、抽象泄漏（你最终还是需要理解 SQL）。

**Query Builder + 类型安全代码生成（推荐）**：
```go
// sqlc：你写 SQL，工具生成类型安全的 Go 代码
const getUser = `SELECT name, email FROM users WHERE id = $1`
// sqlc 自动生成：
func (q *Queries) GetUser(ctx context.Context, id int64) (User, error)
```
SQL 作为真理，代码作为类型安全的壳。

---

## 第四层：缓存——加速重复读取

### 问题

有些查询频繁执行但结果很少变化（比如"热门文章列表"、"用户 session"）。每次都查询数据库是浪费——数据库的磁盘 I/O 是瓶颈。

### 什么是缓存

把查询结果存在**比数据库更快**的地方——通常是**内存**。下次相同查询直接返回缓存数据，不经过数据库。

```python
# 没有缓存
def get_user(user_id):
    return db.query("SELECT * FROM users WHERE id = ?", user_id)
    # 每次调用都查询数据库（磁盘 I/O）

# 有缓存
def get_user(user_id):
    cached = redis.get(f"user:{user_id}")
    if cached:
        return cached                     # 从内存返回（~0.1ms）
    user = db.query("SELECT * FROM users WHERE id = ?", user_id)
    redis.set(f"user:{user_id}", user)    # 存到缓存
    return user                           # （~10ms 省掉了）
```

### 缓存带来新问题——缓存失效

当用户修改了数据，缓存中的旧版本就过时了。你需要决定：
- **设置过期时间**（TTL）：最简单，接受一定时间的"可能不一致"
- **写时主动失效**：数据变更时删除对应缓存。但可能遗漏（代码中有多个写入点）
- **Cache-Aside**（旁路缓存）：应用程序负责维护缓存一致性。最常用的模式

**经验法则**：缓存能不用就不用。只有当瓶颈被证实是重复查询时才加缓存。缓存引入的"数据不一致"bug 比它解决的问题更难排查。

### Redis：不只是缓存

Redis 是后端基础设施中用途最多的组件，远超"缓存"的范畴：

| 用途 | Redis 特性 |
|------|-----------|
| 缓存 | `SETEX key TTL value` — 带过期时间的键值存储 |
| 会话存储 | 存储用户登录状态（多个服务器共享） |
| 限流 | `INCR` + `EXPIRE` — 计数请求频率 |
| 消息队列 | `LPUSH` + `BRPOP` — 简单的生产者-消费者 |
| 分布式锁 | `SETNX` — 在多服务器之间协调"谁来做这件事" |
| 排行榜 | `ZADD` + `ZRANGE` — 有序集合 |

---

## 第五层：从单机到多机——负载均衡的引入

### 问题

一台服务器能处理的并发连接是有上限的。当用户量增长到一台机器撑不住时，你需要在多台机器上运行同一个程序。

### 负载均衡（Load Balancing）

```
                     ┌─▶ Server A (IP: 10.0.0.1)
用户 → 负载均衡器 ──┼─▶ Server B (IP: 10.0.0.2)
                     └─▶ Server C (IP: 10.0.0.3)
```

负载均衡器（Nginx、HAProxy、云厂商的 LB）将收到的请求分发给后端服务器。分发策略：轮询（轮流）、最少连接（发给最闲的）、IP 哈希（同一用户总是到同一台服务器）。

### 多机引入的新问题

**问题 1——会话（Session）**：用户登录时，Session 存在 Server A 的内存中。下一个请求被负载均衡到 Server B，Server B 不认识这个用户。

→ 解决方案：把 Session 存到所有服务器都能访问的地方——Redis。

**问题 2——文件上传**：用户上传的头像保存在 Server A 的磁盘上。下次请求被路由到 Server B 时，头像文件不存在。

→ 解决方案：文件不存服务器本地磁盘，存到**对象存储**（S3、MinIO）。

**关键原则——无状态（Stateless）**：每个服务器实例应该可以随时被销毁和替换，不保存任何"只有这台机器有"的数据。状态（数据库、文件、Session、缓存）全部外移到共享的后端服务。

---

## 第六层：异步处理——请求-响应的瓶颈

### 问题

有些操作的耗时远超 HTTP 请求允许的等待时间。比如发送一封邮件可能需要 3 秒（SMTP 服务器响应慢），生成一份 PDF 报告可能需要 30 秒。用户不会等这么久。

**这些操作不应该在请求-响应的周期中完成。**

### 消息队列：把"确认收到"和"实际处理"分离

```
┌─ 请求-响应路径（毫秒级）────┐    ┌─ 后台处理路径（秒-分钟级）──┐
                              │    │                              │
用户 → 服务器 → 写入消息到队列 →│    │→ Worker 取出消息 → 处理     │
      ← "已收到，处理中"      │    │   （发邮件、生成报告...）    │
                              │    │                              │
```

用户在网站上下单，服务器只做两件事：1) 把订单写入数据库，2) 往消息队列中放入"请发送确认邮件"。然后返回"下单成功"。

另一个进程（Worker）从消息队列中取出消息，调用邮件服务发送。如果邮件发送失败（SMTP 挂了），消息重新入队，稍后重试。

**消息队列的核心价值**：

| 价值 | 说明 |
|------|------|
| **削峰** | 瞬时大量请求不会压垮后端服务（消息在队列中等待） |
| **解耦** | 下单服务不需要知道"怎么发邮件"——它只知道"往队列放一条消息" |
| **重试** | 处理失败的消息可以自动重试，不需要用户重试 |
| **持久化** | 消息写入磁盘，服务器重启不会丢失未处理的消息 |

**主要选择**：

| 消息队列 | 模型 | 适合 |
|----------|------|------|
| **RabbitMQ** | 传统消息代理。消息→交换机→队列→消费者。灵活的路由规则 | 任务队列、可靠投递 |
| **Kafka** | 分布式日志。消息持久化到磁盘，可回放。高吞吐 | 事件流、数据管道、大量数据 |
| **NATS** | 极简、极快、低延迟 | 微服务通信、边缘计算 |
| **Redis Pub/Sub + Stream** | 轻量 | 已有 Redis、简单场景 |

---

## 第七层：认证——谁在访问

### 两个问题

- **认证（Authentication）**：你是谁？证明你是你声称的那个人
- **授权（Authorization）**：你能做什么？在你证明了身份之后，你有哪些权限

### 最常见的方案：Session-based Auth

```
1. 用户用邮箱+密码登录
2. 服务器验证正确 → 创建一个 Session（随机 token），存入 Redis
3. 服务器把 token 作为 Cookie 返回给浏览器
4. 浏览器后续每个请求自动携带 Cookie
5. 服务器从 Cookie 中取出 token，在 Redis 中查找对应的 Session
```

优点：服务器可以随时撤销 Session（删除 Redis 中的记录）。缺点：需要 Redis 存储。

### 无状态方案：JWT

JWT（JSON Web Token）是一个签名过的 JSON 文档。服务器验证签名就知道"这个 token 确实是我签发的，内容没有被篡改过"。

```json
// JWT payload（Base64 编码，不是加密的——任何人都能解码看内容）
{ "user_id": 123, "exp": 1715702400 }
```

优点：不需要服务端存储，适合分布式系统。缺点：无法撤销（除非额外维护黑名单），payload 不能放敏感信息（Base64 编码不是加密）。

### 第三方登录：OAuth 2.0 / OIDC

"用 Google 登录"、"用 GitHub 登录"。OAuth 2.0 是工业标准流程——用户被重定向到 Google 的登录页面，授权后 Google 告诉你的服务器"此人确实是 alice@gmail"。

---

## 第八层：API 设计——前后端之间的合同

前端（浏览器）和后端（服务器）之间的通信需要约定的格式。这就是 API。

### REST（Representational State Transfer）

用 HTTP 方法表示操作，用 URL 表示资源：

```
GET    /users          # 列出所有用户
GET    /users/123      # 获取 ID 为 123 的用户
POST   /users          # 创建新用户
PUT    /users/123      # 完整替换用户 123
PATCH  /users/123      # 部分更新用户 123
DELETE /users/123      # 删除用户 123
```

核心约束：**无状态**——服务器不存储客户端状态。每个请求包含处理它所需的所有信息。

### GraphQL

"取太多"和"取不够"的问题。`GET /users` 返回所有字段（email、phone、address...），但你只需要 name 和 avatar。`GET /users/123` 没有返回 orders，你需要再发一个请求。

GraphQL 让客户端可以精确指定需要哪些字段：
```graphql
query {
  user(id: 123) {
    name
    avatar
    orders(limit: 10) { total amount }
  }
}
```

适用于：复杂的数据关系、多个客户端（iOS 需要的数据和 Web 不同）。

### gRPC

Google 的高性能 RPC 框架。使用 Protocol Buffers（二进制序列化格式，比 JSON 小得多、快得多）。适用于：微服务之间的内部通信（不适用于浏览器直接调用）。

### 选择策略

- **浏览器 ↔ 服务器**：REST（或 GraphQL）
- **服务器 ↔ 服务器**（微服务）：gRPC
- **全栈 TypeScript**：tRPC（端到端类型安全）
- **实时推送**：WebSocket 或 SSE

---

## 第九层：可观测性——当系统变复杂后如何理解它

当后端从"一个程序"变成"几十个服务 + 数据库 + 缓存 + 消息队列 + 队列消费者"之后，"为什么这个请求失败了"变得难以回答。

### 三个支柱

| 支柱 | 回答的问题 | 工具 |
|------|-----------|------|
| **日志（Logging）** | "发生了什么？" 离散的事件记录 | 结构化日志（JSON 格式）、Loki、ELK |
| **指标（Metrics）** | "多少、多快、多少错？" 聚合的数值 | Prometheus + Grafana |
| **链路追踪（Tracing）** | "一个请求经过了哪些服务？" 跨服务追踪 | OpenTelemetry、Jaeger、Tempo |

### 可观测性的最小实践

1. **结构化日志**：不要写 `"用户 Alice 登录成功"`。写 `{"event": "login", "user": "alice", "status": "success"}`。结构化日志可以被查询和聚合
2. **给每个请求分配唯一 ID**：前端生成或在入口服务生成，贯穿所有日志。搜索这个 ID 就能看到这个请求的完整旅程
3. **健康检查**：`/health`（程序是否活着）和 `/ready`（数据库连接是否正常，能否处理请求）

---

## 第十层：部署——从"在我机器上能跑"到用户能访问

### 部署的演进

**阶段 1：手动部署**
```
ssh 到服务器 → git pull → 重启进程
```
简单，但不可复现（服务器上有什么依赖？）。

**阶段 2：Docker 容器**
```dockerfile
FROM python:3.12          # 基础环境
COPY . /app               # 复制代码
RUN pip install -r requirements.txt   # 安装依赖
CMD ["python", "main.py"]
```
把应用和所有依赖打包在一起。在任何装了 Docker 的机器上行为一致。

**阶段 3：Docker Compose（单机多容器）**
```yaml
services:
  app:      # 你的应用
  db:       # PostgreSQL
  redis:    # Redis
  worker:   # 后台 Worker
```
在一台机器上启动所有相关服务。适合中小项目。

**阶段 4：Kubernetes（多机容器编排）**
当一台机器不够用时，需要多台机器协调运行容器。Kubernetes 负责：调度容器到合适的机器、自动扩缩容、负载均衡、滚动更新、服务发现。

**什么时候需要 Kubernetes**：
- 你有 10 个以上需要独立部署和扩缩容的服务
- 你有一个团队专门维护基础设施
- 就中小项目而言，Docker Compose + 一台稍大些的机器通常就够了

**阶段 5：Serverless / FaaS**
根本不关心服务器。上传函数代码，平台在收到请求时自动运行函数，按调用次数和运行时间计费。适合：间歇性负载、事件驱动的场景。

### 部署全景

| 方式 | 复杂度 | 适合 |
|------|--------|------|
| 单机 + systemd | 最低 | 个人项目 |
| Docker + Compose | 低 | 中小项目 |
| PaaS（Fly.io、Railway、Render） | 低 | 不想管基础设施 |
| Kubernetes | 高 | 大规模多服务 |
| Serverless | 中 | 事件驱动、周期性负载 |

---

## 一条请求的旅程

把上面所有层串起来，看一个现实的请求历程：

```
1. 用户点击"下单"

2. 浏览器 → DNS 查询 → 负载均衡器（Nginx / Cloud LB）
   → 负载均衡器把请求转发给 App Server #3

3. App Server #3 的处理逻辑：
   a. 验证 JWT token → 确认用户是 Alice（ID: 123）
   b. 查询 Redis: "Alice 在最近 1 分钟内下了几单" → 限流检查
   c. SELECT ... FROM products WHERE id = 456 → PostgreSQL
   d. BEGIN TRANSACTION
      - INSERT INTO orders ...
      - UPDATE inventory SET stock = stock - 1
      COMMIT
   e. 往 RabbitMQ 发送消息: { "type": "order_confirmed", "order_id": 789 }
   f. 写日志: {"event": "order_created", "user": "alice", "order_id": 789}

4. App Server #3 返回 HTTP 200 {"order_id": 789, "status": "confirmed"}

5. 后台 Worker 从 RabbitMQ 取出消息 → 调用邮件服务发送确认邮件
   → 如果发送失败 → 重试 3 次 → 失败则写入死信队列（人工处理）

6. Prometheus 每 15 秒拉取 App Server 的指标 → Grafana 仪表盘显示
   "过去 5 分钟的订单量"、"平均处理时间"、"错误率"
```

这就是一个现代后端系统的全貌。不是一开始就这么复杂——是从一个进程、一个端口、一个数据库开始，随着用户增长和需求增加，逐一引入这些层次的。

---

## 实用入门路径

### 最小后端程序

选择你熟悉的语言，写一个能响应 HTTP 请求的程序：

```bash
# Go
cat > main.go << 'EOF'
package main
import ("fmt"; "net/http")
func main() {
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, `{"hello":"world"}`)
    })
    http.ListenAndServe(":8080", nil)
}
EOF
go run main.go
# 浏览器打开 http://localhost:8080
```

### 学习路线

1. **理解 HTTP 协议**：请求/响应的结构、方法、状态码。用 `curl -v` 看原始 HTTP 报文
2. **写一个 CRUD API**：对一个资源（比如"任务"）实现增删改查的 HTTP endpoint
3. **接入数据库**：从"数据存内存"到"数据存 PostgreSQL"。理解 SQL 的基本操作
4. **添加认证**：实现注册+登录。理解 bcrypt 密码哈希、JWT 或 Session
5. **理解并发**：你选的框架/语言如何处理多个请求同时到来
6. **容器化**：写 Dockerfile，用 Docker Compose 启动应用+数据库+缓存
7. **学习可观测性**：结构化日志 + Prometheus + Grafana
8. **系统设计**：理解缓存、消息队列、负载均衡解决什么问题

### 关键资源

- **12factor.net**：现代 Web 应用的 12 条设计原则
- **PostgreSQL 官方文档**：数据库文档的黄金标准
- **HTTP 状态码**：httpstatuses.io
- **System Design Primer**（GitHub）：系统设计入门
- **High Performance Browser Networking (Ilya Grigorik)**：理解网络底层
