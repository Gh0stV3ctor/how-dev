# 构建与编译全景——源码怎么变成可运行产物

## 为什么需要"构建"

你写的代码和实际运行的东西之间，隔着一层"翻译"。你写的是给人看的源代码，计算机运行的是机器码或引擎能理解的中间格式。这个翻译过程，就是编译。

但在 Web 开发中，"构建"不只是编译——它还包括打包、优化、代码分割、压缩等等。为什么需要这么多步骤？

**根本原因**：浏览器的能力受限 + 开发效率需要。

| 问题 | 解决方案 |
|------|---------|
| 浏览器只认识 JavaScript，但你想要类型检查 | TypeScript → 编译成 JavaScript |
| 浏览器不认识 JSX（`<div>` 写在 JS 里） | JSX → Babel/SWC 转换为 `React.createElement()` |
| 浏览器不认识 Tailwind class | Tailwind → 扫描代码 → 生成只包含你用过的 class 的 CSS |
| 200 个 `.ts` 文件每个都要下载 → 太慢 | 打包（Bundle）成少数几个文件 |
| 首页加载了整个应用的代码 → 浪费带宽 | 代码分割（Code Splitting）——首页只加载首页的代码 |
| 变量名很长、注释很多 → 浪费传输带宽 | 压缩（Minify）——变量名变短、去掉空格和注释 |

---

## 编译的通用概念

不管什么语言，"编译"的本质相同：

```
源代码（人读的）  ──▶  编译器  ──▶  目标代码（机器/引擎读的）

main.go    ──▶  go build  ──▶  main（ELF 二进制，Linux 可直接执行）
main.rs    ──▶  rustc     ──▶  main（二进制 + 静态链接的系统库）
main.ts    ──▶  tsc       ──▶  main.js（JavaScript，V8 可执行）
main.java  ──▶  javac     ──▶  main.class（JVM 字节码）
main.py    不需要编译（解释型），但可能有 .pyc 字节码缓存
```

**编译型**和**解释型**的区别不在于"是否需要编译"，而在于"编译产物是否可以直接被操作系统执行"：

| 类型 | 编译产物 | 运行方式 | 代表 |
|------|---------|---------|------|
| **AOT 编译**（Ahead-of-Time） | 原生机器码（CPU 可直接执行） | 操作系统直接加载执行 | C、C++、Go、Rust |
| **字节码编译** | 中间格式（需运行时解释） | 运行时 VM 解释 + JIT 编译 | Java（JVM）、C#（.NET CLR） |
| **JIT 编译**（Just-in-Time） | 源码 → 引擎动态编译为机器码 | 运行在引擎内 | JavaScript（V8）、Python（PyPy）、Lua（LuaJIT） |
| **纯解释** | 无编译产物 | 解释器逐行执行 | Python（CPython）、Ruby（CRuby）、Bash |

Go 和 Rust 编译成可直接执行的二进制文件。Java 编译成 `.class`（JVM 字节码），JVM 启动后解释 + JIT 编译。JavaScript 由 V8 引擎在执行时动态编译为机器码。Python 的 CPython 逐行解释（但会生成 `.pyc` 字节码缓存加速后续加载）。

---

## 前端构建流水线

### HTML、CSS、JavaScript——三者到底是什么关系

在讲构建之前，必须先把这三个名词的关系彻底澄清。

**它们不是三种"编程语言"平等排列——它们有根本不同的性质**：

```
HTML       标记语言  →  描述"页面上有什么"（结构）
                        <button>点击</button>
                        
CSS        样式语言  →  描述"这些东西长什么样"（外观）
                        button { color: blue; }
                        
JavaScript 编程语言  →  描述"这些东西怎么动"（行为）
                        button.addEventListener("click", () => { ... })
```

**HTML 不是编程语言**——它没有变量、没有循环、没有条件判断。它只是"标签嵌套标签"来描述文档结构。

**CSS 不是编程语言**——它没有执行逻辑。它只是"选择器 + 属性 + 值"来描述外观。

**JavaScript 是唯一的编程语言**——它可以在浏览器里执行计算、修改 HTML 结构、改变 CSS 样式、发送网络请求。

**三者在页面中的关系**：

```
浏览器加载一个页面时：
1. 下载 HTML → 解析为 DOM 树（页面上有哪些元素）
2. 遇到 <link> → 下载 CSS → 解析为 CSSOM（每个元素对应的样式规则）
3. 遇到 <script> → 下载 JS → 执行
4. JS 执行时可以：
   - 读取/修改 DOM（改 HTML 结构）
   - 读取/修改 CSSOM（改样式）
   - 发送 HTTP 请求（获取数据）
   - 响应用户操作（点击、输入、滚动）

所以不能用"分别完成不同部分"来描述三者的关系。

更准确的说法：
HTML = 蓝图（初始结构），CSS = 装修方案（初始外观），JS = 装修工人（随时可以改蓝图和方案的活人）
```

### JSX 到底是什么——它既不是 HTML 也不是 JS

JSX 是 React 引入的一种**语法扩展**——它让你在 JS/TS 文件中写类似 HTML 的标签。

```jsx
// 你写的 JSX：
const element = <button className="btn">点击</button>;

// 构建工具（Babel / SWC）把它翻译为纯 JavaScript：
const element = React.createElement("button", { className: "btn" }, "点击");

// 运行时 React.createElement() 返回一个 JS 对象：
const element = {
  type: "button",
  props: { className: "btn", children: "点击" }
};
// React 用这个对象来决定创建什么 DOM 节点
```

**JSX 的关键理解**：
- JSX 不是 HTML——它长得像 HTML，但写在 `.js` / `.tsx` 文件里，浏览器不认识它
- JSX 不是一种新语言——它是 JavaScript 的语法糖（`<div>` = `React.createElement("div")`）
- JSX **必须被编译**——构建工具（Babel/SWC）把 JSX 转换为纯 JS 函数调用，浏览器才能执行
- 编译后的产物是纯 JavaScript——所以"用 React 写的页面"在浏览器眼中和"用纯 JS 写的页面"没有区别

**所以前端技术栈的关系应该是这样理解的**：

```
你写的（开发时）                        浏览器收到的（运行时）
─────────────────                      ────────────────
TypeScript (.tsx)    ──编译──▶        纯 JavaScript
  ├── JSX (HTML-ish)  ──编译──▶        React.createElement() 调用
  ├── Tailwind class   ──编译──▶       对应的 CSS 规则
  └── 类型注解         ──擦除──▶       (无——类型在编译后不存在)

项目的 CSS (.css)     ──PostCSS──▶    纯 CSS (只包含你用过的规则)

你不需要手动写 HTML 文件——React 在运行时根据 JS 对象自动生成 DOM 节点。
构建工具自动生成一个最小的 index.html，里面引用编译后的 JS 和 CSS。
```

### 典型的前端构建流水线

这是最复杂的构建流程，因为前端代码是"混搭"的——TypeScript + JSX + Tailwind CSS + 图片 + 字体——而浏览器只认识纯 HTML + CSS + JavaScript。构建过程就是把这些"混搭"的源文件拆解、转换、重新打包成浏览器能直接消费的三种纯产物。

### 典型的前端构建流水线

```
src/
├── app/
│   ├── page.tsx           ← TypeScript + JSX
│   └── globals.css        ← Tailwind CSS
├── components/
│   └── Button.tsx
└── lib/
    └── utils.ts

            │
            ▼  npm run build（Vite / Next.js / Webpack）

1. TypeScript 编译
   page.tsx → tsc 或 swc 做类型检查 + 擦除类型 → 输出纯 JS
   注意：swc 只做语法转换不检查类型，类型检查是 tsc 另做的

2. JSX 转换
   <button onClick={...}>  →  React.createElement("button", {onClick: ...})
   或：→  jsx("button", {onClick: ...})  （新的 JSX Transform）

3. 模块解析
   import { Button } from "@/components/Button"
   → 找到 Button.tsx → 把它也加入编译队列
   → 构建依赖图（谁 import 了谁）

4. Tailwind CSS 生成
   扫描所有 .tsx 文件 → 找到用到的 class（如 bg-blue-500）
   → 只把这些 class 对应的 CSS 规则写入输出文件
   → 你的 globals.css 可能写了很多 Tailwind 指令，但构建产物只有你真正用了的 CSS

5. 打包（Bundle）
   200 个 .ts/.tsx 文件 → 少数几个 JS bundle
   为什么？浏览器同时打开 200 个 HTTP 请求下载 200 个文件太慢了
   一个 HTTP/2 连接可以同时传输，但打包减少的是"发现依赖"的开销

6. 代码分割（Code Splitting）
   首页不需要加载"设置页面"的代码
   → bundle 被拆分为：
     main.js      （首页立即可见的代码）
     settings.js  （用户点击"设置"时才下载）
     vendor.js    （第三方库，可能被缓存很久）

7. 压缩（Minify）
   function calculateTotalPrice(items) { ... }
   → 变量名缩短：function a(b) { ... }
   → 空格删除、注释删除
   → 输出文件的体积可以减少 50-70%

            │
            ▼

build/ 或 .next/static/ 或 dist/
├── assets/
│   ├── index-D4gk9sC8.js        ← 打包+压缩后的 JS
│   ├── index-C1rBNW3d.css       ← 生成的 CSS
│   └── logo-abc123.svg          ← 哈希命名的静态资源
├── chunks/                      ← 代码分割出的各个 chunk
└── index.html                   ← 入口 HTML（引用上面的 JS/CSS）
```

### 构建工具不是"编译器"——它们是"流水线管理器"

Vite、Webpack、Turbopack 本身不编译 TypeScript、不转换 JSX、不生成 Tailwind。它们是一个**调度平台**——按顺序调用各个"子工具"。

```
Vite 的构建过程：
  Vite 遇到 .tsx 文件 → 调用 esbuild（或 swc）做 TS → JS + JSX → JS
  Vite 遇到 .css 文件 → 调用 PostCSS（含 Tailwind 插件）生成最终 CSS
  Vite 遇到 .svg 文件 → 作为静态资源复制到输出目录
  Vite 遇到 import("./heavy.js") → 标记为代码分割点

Webpack 的构建过程：
  同 Vite，但 Vite 用 esbuild（Go 写的，快）而 Webpack 用 babel（JS 写的，慢）。
  开发时 Vite 用原生 ESM（浏览器直接 import，不再打包），Webpack 永远先打包。
```

### 开发模式 vs 生产构建

| | `npm run dev` | `npm run build` |
|------|-------------|----------------|
| **目的** | 快速反馈——改代码立刻看到效果 | 优化产物——给用户最小的下载量 |
| **编译方式** | 按需（当前页面涉及的文件才编译） | 全部（构建整个项目的依赖图） |
| **输出** | 内存中（不写磁盘），通过 dev server 提供 | 输出到磁盘（`.next/static/` 或 `dist/`） |
| **优化** | 不做（保持构建速度快） | 压缩、树摇、代码分割 |
| **Source Map** | 内联（方便调试，但文件大） | 外部文件（不发给用户，但支持错误追踪） |
| **HMR** | 热模块替换——改一个组件，不刷新页面就更新 | 不适用——这是最终产物 |

---

## 后端编译流水线

后端编译比前端简单，因为"目标环境"更清晰——就是一个运行在服务器上的进程。

### Node.js 后端

```
src/
├── routes/
│   └── users.ts          ← TypeScript
├── middleware/
│   └── auth.ts
└── index.ts

    │
    ▼  npm run build（tsc 或 tsx）

dist/
├── routes/
│   └── users.js          ← 编译产物：JavaScript（Node.js 可执行）
├── middleware/
│   └── auth.js
└── index.js

    │
    ▼  node dist/index.js
    │
    ▼
进程启动，监听 localhost:3001，等待请求
```

Node.js 后端的编译和前端 TypeScript 编译本质相同——`tsc` 把 TypeScript 转成 JavaScript。区别在于：

- 前端 JS 打包成一个文件，后端 JS 保持目录结构（因为 Node.js 有 `require()` / `import` 可直接加载不同文件）
- 前端的目标环境是浏览器 V8，后端的目标环境是 Node.js V8——引擎版本不同，可用的 API 不同
- 后端编译产物**不压缩**、**不打包**（文件在服务器本地，不通过慢速网络传输）

### Go 后端

```
src/
├── cmd/
│   └── server/
│       └── main.go        ← 入口
├── internal/
│   ├── handler/
│   └── store/
└── go.mod

    │
    ▼  go build -o server ./cmd/server/

server                     ← 一个文件！单一二进制
                             ├── Go 标准库（静态链接）
                             ├── 第三方库代码
                             └── 你的业务代码

    │
    ▼  ./server
    │
    ▼
进程启动，监听 localhost:3001，等待请求
```

Go 编译的特殊性：

- 编译产物是**一个文件**——不需要 Node.js 运行时，不需要 `node_modules/`
- 静态链接：Go 的二进制文件包含了所有依赖，包括 Go 标准库（除极少数 C 库调用）
- 交叉编译：`GOOS=linux GOARCH=amd64 go build` → 在你的 Mac/Windows 上编译出 Linux 可执行的二进制
- 编译速度极快：Go 编译器从头设计的，增量编译 + 轻量语法 = 大型项目几秒编译完成

### Rust 后端

```
cargo build --release
    │
    ▼
target/release/server      ← 单一二进制
                             ├── 静态链接的 Rust 标准库 + 第三方 crate
                             └── 你的业务代码

区别：
- Rust 编译器基于 LLVM，编译慢（大型项目几分钟到几十分钟）
- 但产物性能极高（和 C/C++ 同级，零运行时开销）
- 交叉编译需要配置目标工具链（比 Go 复杂）
```

### Python 后端

```
Python 不需要编译步骤。

main.py → python main.py → 进程启动，监听端口
```

但 Python 有相关的概念：

- `.pyc` 字节码：CPython 解释器会在第一次导入时把 `.py` 编译为 `.pyc`（字节码），缓存在 `__pycache__/` 目录。再次导入时直接加载 `.pyc`，跳过解析步骤
- 分发工具：PyInstaller 可以把 Python 项目打包成一个独立可执行文件（内含 Python 解释器 + 所有依赖）——但体积大（几十 MB 起步）
- Cython：把 Python 代码编译成 C 扩展（`.so`），既能提速又能隐藏源码
- Nuitka：把 Python 编译成 C，再编译为机器码

### Java 后端

```
src/
└── main/java/com/example/
    └── Application.java

    │
    ▼  ./gradlew build 或 mvn package

build/libs/app.jar        ← JAR 文件（Java Archive）
                             ├── .class 字节码文件
                             └── 依赖信息

    │
    ▼  java -jar app.jar
    │
    ▼
JVM 启动 → 加载 .class → 解释 + JIT 编译 → 进程运行
```

Java 的特殊性：

- 编译产物是 JVM 字节码（`.class`），不是机器码——需要 JVM 运行时
- JVM 运行时做 JIT（Just-in-Time）编译——热点代码（频繁执行的）动态编译为机器码，冷代码继续解释
- JAR 是打包格式——把几百个 `.class` 文件打包成一个 `.jar`

---

## 文件后缀速查

| 后缀 | 语言 | 编译工具 | 编译产物 | 运行环境 |
|------|------|---------|---------|---------|
| `.ts` | TypeScript | tsc / swc / esbuild | `.js` | Node.js 或浏览器 |
| `.tsx` | TypeScript + JSX | tsc / swc / esbuild + JSX transform | `.js` | 浏览器 |
| `.jsx` | JavaScript + JSX | Babel / swc | `.js` | 浏览器 |
| `.js` | JavaScript | 通常不需要编译 | `.js` | Node.js 或浏览器 |
| `.go` | Go | `go build` | 单一二进制文件 | 操作系统原生 |
| `.rs` | Rust | `rustc` / `cargo build` | 单一二进制文件 | 操作系统原生 |
| `.py` | Python | 不需要（但 `.pyc` 缓存） | `.pyc`（可选） | CPython 解释器 |
| `.java` | Java | `javac` | `.class`（JVM 字节码） | JVM |
| `.kt` | Kotlin | `kotlinc` | `.class`（JVM）或原生二进制 | JVM 或操作系统原生 |
| `.swift` | Swift | `swiftc` | 单一二进制文件 | 操作系统原生（Apple 平台） |
| `.cpp` / `.cc` | C++ | `g++` / `clang++` | 单一二进制或 `.o` + 链接 | 操作系统原生 |
| `.c` | C | `gcc` / `clang` | 单一二进制或 `.o` + 链接 | 操作系统原生 |
| `.scss` / `.less` | SCSS/Less | sass / less | `.css` | 浏览器 |
| `.css` | CSS | PostCSS / Tailwind | `.css` | 浏览器 |
| `.html` | HTML | 不需要编译 | `.html` | 浏览器 |

---

## 下一章

源码变成了可运行的产物——前端产物在 `.next/static/`（或 `dist/`），后端产物在 `.next/server/`（或二进制文件）。下一步要问：

**这些产物放在一起还是分开放？前后端代码是一个 Git 仓库还是两个？**

→ [03 项目组织](03-project-organization.md)
