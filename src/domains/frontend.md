# 前端开发全景指南

> **前置阅读**：[JavaScript / TypeScript 全景指南](../languages/javascript.md) — 本文假设你已了解 JS/TS 语言基础。

---

## 技术全景图：各层之间的关系

前端不是一堆工具的堆砌，而是一个有明确层级依赖的技术栈。每一层解决上一层没解决的问题：

```
┌────────────────────────────────────────────────┐
│  组件库 (shadcn/ui, Ant Design)                 │  ← 预制 UI，不重复造轮子
│  依赖：框架                                      │
├────────────────────────────────────────────────┤
│  元框架 (Next.js, Nuxt, SvelteKit)              │  ← 路由、SSR、数据加载、构建整合
│  依赖：UI 框架 + 打包器                          │
├────────────────────────────────────────────────┤
│  CSS 方案 (Tailwind, CSS Modules)   状态管理     │  ← 样式组织      共享数据
│  依赖：框架                         依赖：框架    │
├────────────────────────────────────────────────┤
│  UI 框架 (React, Vue, Svelte, Solid)            │  ← 组件化 + 响应式渲染
│  依赖：JavaScript + 浏览器 API                  │
├────────────────────────────────────────────────┤
│  TypeScript                                     │  ← 静态类型检查
│  依赖：JavaScript                               │
├────────────────────────────────────────────────┤
│  JavaScript + 浏览器 API (DOM, Fetch, ...)       │  ← 语言 + 平台能力
├────────────────────────────────────────────────┤
│  打包器 (Vite, Webpack, Turbopack)              │  ← 编译、打包、优化
│  依赖：所有需要编译的上层                        │
└────────────────────────────────────────────────┘
```

**关键理解**：每一层都是**为了解决上一层存在的具体痛点而诞生的**，不是"有比没有好"的可选配件。下面自底向上解释每一层。

---

## 第一层：裸 JavaScript + 浏览器——没有框架的世界

### 浏览器给了你什么

JavaScript 在浏览器中运行时，可以访问一组浏览器提供的 API。这是前端开发最底层的"原材料"：

| API | 你能做什么 |
|-----|-----------|
| **DOM API** (`document.getElementById`, `createElement`) | 读取、修改、添加、删除页面上的元素 |
| **Event API** (`addEventListener`) | 响应用户操作：点击、输入、滚动 |
| **Fetch API** (`fetch("/api/users")`) | 向服务器发 HTTP 请求，获取或提交数据 |
| **History API** (`pushState`) | 无刷新地改变浏览器地址栏的 URL |
| **CSSOM** (`element.style`, `classList`) | 在 JS 中操控元素样式 |

有了这些，你就能写出一个完整的交互式应用。不需要任何框架——浏览器本身就足够了。

### 一个完整的、没有框架的交互页面

这是一个在页面上显示计数、点击按钮增加计数的例子：

```html
<!DOCTYPE html>
<html>
<body>
  <button id="btn">点击次数: 0</button>

  <script>
    // 第一步：从页面上找到按钮这个元素（DOM API）
    const button = document.getElementById("btn");

    // 第二步：声明一个变量来存储"当前计数"——这就是"状态"
    let count = 0;

    // 第三步：写一个函数，把"当前计数"反映到页面上
    function updateUI() {
      button.textContent = `点击次数: ${count}`;
    }

    // 第四步：监听点击事件，点击时更新状态并刷新界面（Event API）
    button.addEventListener("click", () => {
      count = count + 1;      // 改变状态
      updateUI();              // 手动刷新界面
    });
  </script>
</body>
</html>
```

这段代码揭示了前端开发最核心的问题：**状态和界面是分开的**。

- `count` 是**状态**（数据），存于 JavaScript 变量中
- `button.textContent` 是**界面**，存于 DOM 中
- 每当 `count` 发生变化，你必须**自己写代码**把变化反映到 DOM 上——调用 `updateUI()`

当你的应用只有 3 个变量和 3 个 DOM 元素时，这完全可控。当有 300 个变量和 300 个 DOM 元素，而且它们互相依赖——用户名变了影响导航栏、侧边栏、评论区三个地方——手动追踪"哪些 DOM 需要更新"就变成了噩梦。

**这就是 UI 框架要解决的问题。**

---

## 第二层：UI 框架——让界面自动跟随数据

### 框架做了什么

如果你一直在写 `count = count + 1; updateUI();`，你会想："我能不能只写 `count = count + 1`，框架自动帮我把界面更新了？"

这就是 UI 框架的核心价值：**你只管修改数据，框架负责更新界面。**

用 React 重写上面的例子：

```jsx
// 这段代码在 App 组件函数内部
const [count, setCount] = useState(0);

return (
  <button onClick={() => setCount(count + 1)}>
    点击次数: {count}
  </button>
);
```

分解这段代码，理解框架做了什么：

```jsx
// useState(0)：声明一个状态变量，初始值为 0
//     返回两个东西：
//     - count：当前状态的值（第一次是 0）
//     - setCount：修改状态的函数（调用它会触发重新渲染）
const [count, setCount] = useState(0);

// return (...) 是 JSX——看起来像 HTML 但实际是 JavaScript。
//     浏览器不认识 JSX，它是"语法糖"。
//     构建时被 Vite 转换为 JavaScript 函数调用。
//     浏览器最终执行的代码是：
//     React.createElement("button", {onClick: ...}, "点击次数: " + count)
return (
  // onClick 里调用 setCount：状态 +1，然后 React 自动重新执行组件函数，
  //     用新的 count 值生成新的 JSX，React 找出 DOM 的最小差异并更新
  <button onClick={() => setCount(count + 1)}>
    点击次数: {count}
  </button>
);
```

**这是理解 React 最关键的一点**：

1. 你调用 `setCount(新值)` —— 表示"状态变了"
2. React 重新执行整个 `App()` 函数 —— 用新值生成全新的 JSX
3. React 把新旧 JSX 做比较（diffing），计算出**最小差异**——比如"只有 button 里的文字变了，不需要重建 button 节点"
4. React 把最小差异应用到真实 DOM 上

所以你对框架的认知应该是：**你用 JSX 声明"这个界面应该长这样（作为数据的函数）"；当数据变化时，框架重新执行声明、计算差异、精确更新 DOM。你不再写 `document.getElementById` 和 `textContent =`。**

### 三大框架的不同实现

虽然目标相同（数据驱动界面自动更新），但实现方式不同：

**React**：状态变化 → 重新生成 Virtual DOM → 与旧 Virtual DOM diff → 更新真实 DOM。每次状态更新时，组件函数重新执行。

**Vue**：状态变化 → 响应式系统精确知道"这个状态变量被哪些 DOM 节点依赖" → 直接更新那些节点。`ref()` 包装的变量被修改时，只有用到它的地方才重新渲染。

**Svelte**：构建时分析模板 → 编译生成直接操作 DOM 的代码。运行时没有"框架"在运行——只有你自己写的逻辑。

```svelte
<!-- Svelte：看起来只是 HTML 扩展，实际上 count 是响应式的 -->
<script>
  let count = 0;
  // Svelte 编译器看到 {count}，知道这个 span 依赖于 count
  // 编译后生成代码：count 变化时，直接更新这个 span 的 textContent
</script>

<button on:click={() => count++}>
  点击次数: {count}
</button>
```

**内核总结**：框架做的事情就是把 `count = count + 1; updateUI()` 的后半句自动化了。你不再手动更新 DOM，而是声明"界面是数据长这样的函数"，框架来做剩下的。

---

## 第三层：CSS 方案——样式怎么组织

框架解决了"界面如何跟随数据自动更新"，但没有解决"界面怎么好看"。你需要 CSS。

### 最朴素的方式及其问题

```css
/* styles.css — 全局 CSS */
.button { background: blue; color: white; }
```

一个项目有几十个开发者时，所有人都往 `styles.css` 里加东西。迟早有两个人的 `.button` 互相覆盖——而且你很难排查是"谁的规则生效了"。

### 主流的解决方案

| 方案 | 怎么解决名字冲突 | 代码长什么样 |
|------|----------------|------------|
| **CSS Modules** (`Button.module.css`) | 编译时给类名加哈希（`.button` → `.button_a3f2`），自动作用域隔离 | `.button { ... }` → `import styles from "./Button.module.css"` → `<button className={styles.button}>` |
| **Tailwind CSS** | 不需要命名——样式直接写在 class 属性里，每个 class 做一件事 | `<button class="bg-blue-500 text-white rounded px-4 py-2">` |
| **CSS-in-JS** (styled-components, Panda CSS) | 样式绑定在组件上，天然隔离。运行时方案有性能开销，零运行时方案编译时提取为 CSS | `const Button = styled.button\`...\` ` |

**Tailwind 已经成为新项目最主流的选择**。它的哲学：与其费劲想"这个按钮应该叫 `primary-button` 还是 `submit-btn`"，不如直接用预定义的、功能单一的工具类组合。

---

## 第四层：状态管理——组件之间的数据共享

框架给了每个组件自己的 `useState`。但当两个组件需要共享同一份数据时呢？

### 问题场景

```
     [Navbar] ← 需要显示"当前用户名"
        │
     [SettingsPage] ← 用户在这里可以修改用户名
        │
     用户名被修改后，Navbar 怎么知道要更新？
```

### 解决层次

**第一级：提升到共同父组件**。把状态放在 Navbar 和 SettingsPage 的共同父组件中，通过 props 向下传递。简单场景够用。

**第二级：Context / Provider**。当数据需要跨越多层中间组件（它们可能不关心这个数据），用 Context 避免每一层都手动传递 props。

**第三级：全局状态**。当应用规模变大、多个完全不相关的页面需要共享数据时，使用专门的状态管理库：

| 工具 | 适配框架 | 核心思想 |
|------|---------|---------|
| **Zustand** | React | 极简——一个函数创建一个 store，组件直接 `useStore(state => state.xxx)` |
| **Pinia** | Vue | Vue 官方——类似 Zustand 的理念，`defineStore` 定义，组件中 `store.xxx` 直接读写 |

### 你不应该放进全局状态的数据

**从服务器获取的数据**（用户列表、文章内容）不是"应用状态"——它们是**服务端数据的客户端缓存**。管理这类数据需要处理：缓存过期、后台刷新、乐观更新、重试。这有专门的工具：

```typescript
// TanStack Query（React）/ Vue Query 管理服务端数据
const { data: users, isLoading, error } = useQuery({
  queryKey: ["users"],
  queryFn: () => fetch("/api/users").then(res => res.json()),
});
// 自动缓存 /users 的响应
// 组件重新挂载时使用缓存而非重新请求
// 后台自动刷新过期数据
```

**区分 UI 状态和服务端缓存**——这是前端状态管理最重要的判断。

---

## 第五层：元框架——把路由、SSR、数据加载、构建整合在一起

### UI 框架没做什么

React/Vue/Svelte 只管"数据和界面之间的关系"。它们不管：

- **路由**：用户在地址栏输入 `/settings` 时，显示哪个组件？
- **服务端渲染（SSR）**：在服务器上生成完整 HTML，让用户看到页面更快
- **数据加载**：进入某个路由之前先获取数据
- **构建配置**：生产环境的打包优化

这些你可以自己拼：React + React Router + 手动配置 Vite 的 SSR。但每个项目都重新拼一遍是浪费时间。

**元框架把这些"框架没做的事"整合成一体的方案。** 它是 UI 框架之上的"应用框架"。

### 元框架做了什么（以 Next.js 为例）

一个 Next.js 项目的基本结构：

```
myapp/
├── app/                   # App Router（Next.js 的文件系统路由）
│   ├── layout.tsx         #   整个应用的"外壳"（导航栏、页脚）
│   ├── page.tsx           #   对应 URL "/"
│   ├── settings/
│   │   └── page.tsx       #   对应 URL "/settings"
│   └── api/
│       └── users/
│           └── route.ts   #   API 端点（后端逻辑，同一个项目中）
├── next.config.ts         #   Next.js 配置
├── package.json
└── tsconfig.json
```

**文件系统路由**：`app/settings/page.tsx` 这个文件自动对应 URL `/settings`。你不需要写路由配置——文件的位置就是路由。

**React Server Components（RSC）——服务器端组件不需要发 JS**：

```tsx
// app/page.tsx — 默认是 Server Component（在服务器端运行，不发送 JS）
// 这个组件可以直接查询数据库，因为它在服务器上运行
import { db } from "@/lib/db";

export default async function Page() {
  const posts = await db.query("SELECT * FROM posts ORDER BY created_at DESC");
  return (
    <ul>
      {posts.map(post => <li key={post.id}>{post.title}</li>)}
    </ul>
  );
}
```

这个组件的代码永远不会到达浏览器。它在服务器上执行，生成完整的 HTML。浏览器收到的是纯 HTML，**零 JavaScript**。

```tsx
// 需要交互的组件才标记 "use client"
"use client";
import { useState } from "react";

export function LikeButton() {
  const [liked, setLiked] = useState(false);
  return <button onClick={() => setLiked(!liked)}>{liked ? "♥" : "♡"}</button>;
}
```

**核心理解**：Next.js App Router 的默认是"服务端运行"。只有在你显式标记 `"use client"` 时才发送 JavaScript。目标是**尽可能少地发送 JS 到浏览器**。

### 主要元框架对比

| 元框架 | 基于 | 路由方案 | 特色 |
|--------|------|---------|------|
| **Next.js** | React | 文件系统路由（App Router） | RSC、Vercel 生态、最大社区 |
| **Nuxt** | Vue | 文件系统路由 | Vue 的 Next.js 等价物 |
| **SvelteKit** | Svelte | 文件系统路由 | Svelte 官方方案、适配器架构 |
| **Remix** | React | 文件系统路由（扁平） | 拥抱 Web 标准（Web Fetch API、FormData） |
| **Astro** | 多框架 | 文件系统路由 | 默认零 JS、内容型站点首选 |

---

## 第六层：组件库——不重写按钮、对话框、表格

元框架给了应用骨架。但当你要写一个对话框（Dialog）、下拉菜单（Dropdown）、日期选择器（DatePicker）时，你不需要从零写。

**组件库给你预制的、可定制的 UI 组件。**

| 类型 | 代表 | 特点 |
|------|------|------|
| **可复制（代码即你的）** | shadcn/ui | 组件源码直接复制到你的项目中，完全可控。依赖 Tailwind + Radix |
| **完整设计系统** | Ant Design, MUI | 开箱即用的完整组件库，有统一的视觉语言 |
| **无样式基础组件** | Radix UI, Headless UI | 只处理行为（点击、展开、键盘导航），不管样式——你自己加 |

**shadcn/ui 为什么流行**：它不是一个 npm 包——你运行 `npx shadcn-ui add button`，它把 Button 组件的源码直接写入你的 `src/components/` 目录。你可以修改任何东西。这击中了组件库最大的痛点："我想要这个组件 90% 的功能，但 10% 需要定制"。

---

## 所有层的关系：以一个小项目为例

假设你在做一个任务管理应用。用户打开后看到任务列表，可以添加任务、标记完成。

**项目初始化**：

```bash
npm create vite@latest taskapp -- --template react-ts
```

这一条命令创建了什么？它配置好了打包器（Vite）、框架（React）、类型系统（TypeScript）的组合。创建的项目结构：

```
taskapp/
├── src/
│   ├── main.tsx           # 应用入口——React 从这里开始挂载到 HTML 上
│   └── App.tsx            # 根组件——应用的起点
├── index.html             # HTML 入口——浏览器最先加载的文件
├── tsconfig.json          # TypeScript 配置
├── vite.config.ts         # Vite 配置（打包器）
└── package.json           # 依赖声明
```

**关键跟踪：从浏览器到你的代码，经过什么路径**：

```
1. 浏览器加载 index.html
   → 里面有一个 <div id="root"></div> 和 <script src="/src/main.tsx">

2. Vite 开发服务器拦截了对 /src/main.tsx 的请求
   → 在内存中编译 TypeScript + JSX → 转换成浏览器能理解的 JavaScript
   → 发给浏览器

3. main.tsx 执行：
   import App from "./App";
   ReactDOM.createRoot(document.getElementById("root")).render(<App />);
                              ↑ 把 React 组件树挂载到 HTML 的 <div id="root"> 上

4. App.tsx 是应用的根组件，从这里开始整个 React 组件树
```

---

## 回到实战：搭建项目的基础工作流

以下是一个前端项目的典型生命周期。每一步都用具体的命令和解释说明：

### 第一步：用打包器脚手架初始化

```bash
# Vite 脚手架创建项目，给你开箱即用的配置
npm create vite@latest myapp -- --template react-ts
cd myapp
npm install
```

**解释**：`create vite` 是 Vite 的项目模板生成器。它从模板仓库拉取文件、写入你的项目目录。`--template react-ts` 选择了"React + TypeScript"模板。这个模板为你准备好了 Vite 配置、TypeScript 配置、入口文件和基础的 App 组件。你不用自己写任何配置。

`npm install` 读取 `package.json` 中的依赖列表，下载并安装到 `node_modules/` 目录。

### 第二步：启动开发服务器

```bash
npm run dev
# 输出: VITE v5.x.x  ready in 300ms
#       ➜ Local: http://localhost:5173/
```

**解释**：`npm run dev` 执行 `package.json` 中 `scripts.dev` 定义的命令（通常是 `vite`）。Vite 启动一个开发服务器，监听 5173 端口。当你用浏览器打开 `http://localhost:5173` 时：

1. Vite 返回 `index.html`
2. 解析 `<script src="/src/main.tsx">`
3. Vite 在内存中编译 `main.tsx`（TypeScript → JavaScript，JSX → `createElement`），返回给浏览器
4. 浏览器执行编译后的 JavaScript
5. React 启动，挂载到页面上

Vite 在文件修改后，通过 **HMR（Hot Module Replacement，模块热替换）** 只重新编译修改了的文件并推送给浏览器——不需要刷新页面，状态不丢失。

### 第三步：添加路由（用元框架替掉裸 Vite + 手写路由）

如果你的应用有多个页面（首页、设置页、个人资料页），你需要路由。如果你选了 Next.js：

```bash
npx create-next-app@latest myapp
# 这个命令创建了一个 Next.js 项目，替代 Vite 的脚手架
```

**为什么从 Vite 换到 Next.js**：Next.js 内置了路由、服务端渲染、代码分割。你不需要手动集成 React Router、手动配置 SSR。

### 第四步：日常开发循环

```bash
# 写代码
nvim src/app/page.tsx     # 编辑页面

# 浏览器自动更新（HMR）
# 在浏览器 devtools 中查看网络请求、console、React 组件树

# 提交前检查
npm run build             # 生产构建（检查类型、打包、优化）

# 如果构建通过：
git add . && git commit -m "..."
```

### 第五步：部署

```bash
npm run build
# 产出目录通常是 dist/（Vite）或 .next/（Next.js）
# dist/ 里的文件是纯静态的 HTML + CSS + JS + 图片
# 部署到任何静态托管服务：
# Vercel: 连 GitHub 仓库，推送即部署
# Netlify: 同上
# 自托管 Nginx: 把 dist/ 目录复制到服务器上的 /var/www/
```

---

## 选型指南：什么场景选什么方案

| 你的项目是... | 推荐 |
|-------------|------|
| 个人博客、文档站、营销页面 | Astro + Tailwind |
| 需要登录、复杂交互的 Web 应用 | Next.js（React）或 Nuxt（Vue） + Tailwind + shadcn/ui |
| 只需要一个简单的交互页面（不需要路由） | Vite + React/Vue + Tailwind |
| 从零学习前端 | HTML + CSS + 原生 JS（理解浏览器平台本身）→ 然后引入框架 |
| 已有 React 项目需要添加路由和 SSR | 迁移到 Next.js |
| 已有 Vue 项目需要添加路由和 SSR | 迁移到 Nuxt |

---

## 实用入门路径

```bash
# 最短路径：5 分钟跑起来
npm create vite@latest hello -- --template react-ts
cd hello && npm install && npm run dev
# 浏览器打开 localhost:5173，你就在开发前端了
```

学习路线：
1. **理解浏览器平台**：DOM 是什么、事件是什么、CSS 的盒模型
2. **用原生 JS 写一个交互页面**（计数器、ToDo 列表）——体会框架要解决的问题
3. **引入框架**（React 或 Vue）：理解 state、props、组件
4. **用 Tailwind** 让页面好看
5. **引入元框架**（Next.js 或 Nuxt）：理解路由和 SSR
6. **做一个完整的 side project**——这是唯一真正学会的方法

关键资源：
- **MDN Web Docs**：developer.mozilla.org，Web 技术权威参考
- **react.dev / vuejs.org**：框架官方文档，质量都极高
- **Vite 文档**：vitejs.dev
- **Next.js 教程**：nextjs.org/learn
