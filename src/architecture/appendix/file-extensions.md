# 文件后缀速查

## 前端相关

| 后缀 | 语言/格式 | 编译/处理工具 | 编译产物 | 运行环境 | 典型用途 |
|------|----------|-------------|---------|---------|---------|
| .html | HTML | 无 | .html | 浏览器 | 页面结构 |
| .css | CSS | PostCSS, Sass, Less | .css | 浏览器 | 页面样式 |
| .scss | SCSS | Sass (dart-sass) | .css | 浏览器 | CSS 预处理 |
| .less | Less | Less 编译器 | .css | 浏览器 | CSS 预处理 |
| .js | JavaScript | 通常不需要 | .js | 浏览器或Node.js | 前端逻辑或后端 |
| .mjs | ES Module JS | 无 | .mjs | Node.js(ESM)或浏览器 | ESM 模块 |
| .cjs | CommonJS | 无 | .cjs | Node.js(CJS) | CJS 模块 |
| .ts | TypeScript | tsc/swc/esbuild | .js | 浏览器或Node.js | JS+类型系统 |
| .tsx | TS+JSX | tsc+JSX transform | .js | 浏览器 | React组件(TS) |
| .jsx | JS+JSX | Babel/swc | .js | 浏览器 | React组件(JS) |
| .vue | Vue组件 | @vue/compiler-sfc | .js+.css | 浏览器 | Vue SFC |
| .svelte | Svelte组件 | Svelte编译器 | .js+.css | 浏览器 | Svelte组件 |
| .json | JSON | 无 | .json | 任何 | 数据/配置 |
| .svg | SVG | SVGO(可选优化) | .svg | 浏览器 | 矢量图标 |
| .md/.mdx | Markdown/MDX | 内容框架 | HTML/JSX | 浏览器 | 文档内容 |

## 后端相关

| 后缀 | 语言 | 编译工具 | 编译产物 | 运行环境 |
|------|------|---------|---------|---------|
| .go | Go | go build | 单一二进制 | 操作系统原生 |
| .rs | Rust | rustc/cargo build | 单一二进制 | 操作系统原生 |
| .py | Python | 无需(解释型), .pyc缓存 | .pyc | CPython |
| .java | Java | javac | .class→.jar | JVM |
| .kt | Kotlin | kotlinc/Kotlin Native | .class/.jar或原生 | JVM或原生 |
| .swift | Swift | swiftc | 单一二进制 | Apple平台 |
| .c | C | gcc/clang | .o→链接→二进制 | 操作系统原生 |
| .cpp/.cc | C++ | g++/clang++ | .o→链接→二进制 | 操作系统原生 |
| .rb | Ruby | 无需(解释型) | 无 | CRuby |
| .php | PHP | 无需 | 无 | PHP解释器(PHP-FPM) |
| .cs | C# | dotnet build | .dll(CIL) | .NET CLR |
| .ex/.exs | Elixir | mix compile | BEAM字节码 | BEAM VM |
| .lua | Lua | 无需或luac | 字节码 | Lua解释器 |

## 配置和构建

| 文件 | 什么工具 | 做什么 |
|------|---------|------|
| package.json | Node.js/npm/Vite | 项目元数据、依赖、脚本 |
| tsconfig.json | TypeScript(tsc) | TS编译选项 |
| next.config.ts | Next.js | Next.js构建选项 |
| vite.config.ts | Vite | Vite构建和插件 |
| tailwind.config.ts | Tailwind CSS | 主题和内容路径 |
| go.mod | Go | Go依赖声明 |
| go.sum | Go | Go依赖锁定(自动生成) |
| Cargo.toml | Rust(Cargo) | Rust依赖声明 |
| Cargo.lock | Rust(Cargo) | Rust依赖锁定(自动生成) |
| pyproject.toml | Python | Python统一配置 |
| Dockerfile | Docker | Docker镜像构建说明 |
| docker-compose.yml | Docker Compose | 多容器编排 |
| Makefile | make | 通用构建脚本 |
| .env/.env.local | 各种框架 | 环境变量(不提交Git) |
| .gitignore | Git | 指定不进入版本控制的文件 |
| nginx.conf | Nginx | Nginx服务器配置 |
