# How Dev — 开发者技术全景指南

## 这是什么

面对一个空白编辑器和新语言/技术栈时，第一步不是学语法——而是理解这片领域的"地图"：工具链关系、生态位、编码惯例、从源码到产物的完整路径。

本仓库提供一系列**技术全景文档**，分为三层：

- **topics/** — 先读。编程世界的通用概念地图（语言有哪些属性维度？编译/解释/JIT 三条路径各是什么？工具链里有哪些生态位？）
- **languages/** — 后读。每门语言在这些概念维度上做了什么选择，以及该语言特有的工具和惯例
- **domains/** — 跨语言领域指南。从零理解前端、后端、系统架构、开发生态全貌与现状

## 文档结构

```
how-dev/
├── requirements.md                      # 项目需求文档与设计决策
├── topics/                              # 先读：跨语言通用概念地图
│   ├── 01-language-profile.md           # 语言画像：执行模型/类型系统/内存管理/范式的分类学
│   ├── 02-build-pipeline.md             # 从源码到运行：编译型/字节码VM/JIT 三条路径的完整管线
│   ├── 03-toolchains.md                 # 工具链生态位：编译器/构建系统/linter/formatter/LSP/调试器的全景
│   ├── 04-package-management.md         # 包管理通识：依赖解析、锁文件、注册中心、版本约束
│   ├── 05-project-structure.md          # 项目结构约定：各语言社区的目录布局模式对比
│   ├── 06-coding-idioms.md              # 编码惯用法：错误处理/资源管理/并发模型/空值处理的方案谱系
│   ├── 07-testing.md                    # 测试版图：单测/mock/benchmark/fuzz/覆盖率的跨语言对比
│   └── 08-deployment.md                 # 部署与分发：产物形态/交叉编译/发布渠道的跨语言对比
├── languages/                           # 后读：按语言的全景指南
│   ├── c.md                             # C：编译型/手动内存/过程式——理解计算机的基础语言
│   ├── cpp.md                           # C++：零成本抽象/RAII/模板——控制一切的系统语言
│   ├── python.md                        # Python：动态类型/字节码VM——AI时代的大赢家
│   ├── go.md                            # Go：goroutine并发/单一二进制——云基础设施的标准语言
│   ├── rust.md                          # Rust：所有权系统/零成本安全——最受尊敬的下一门语言
│   ├── javascript.md                    # JS/TS：浏览器原生/JIT——唯一跑遍四个平台的语言
│   └── typst.md                         # Typst：标记+脚本——LaTeX 的现代替代
└── domains/                             # 跨语言领域指南
    ├── frontend.md                      # 前端全景：从裸 JS 到元框架的六层技术栈
    ├── backend.md                       # 后端全景：从 TCP Socket 到 Serverless 的十层演进
    ├── system-architecture.md           # 系统架构：客户端-服务器模式在进程/OS/网络每一层
    ├── development-ecosystem.md         # 开发生态大地图：语言→框架→元框架→组件库→基础设施的概念分类
    ├── ecosystem-state.md               # 生态现状报告（2026）：谁在主导、谁在上升、该学什么
    └── reference-projects.md            # 全栈参考项目对比：cal.diy/Plane/Lemmy 等优质 OSS 代码库
```

## 使用方式

**从头了解一个新语言**：先读 `topics/` 中对应的概念文档建立框架，再读 `languages/` 中该语言的文档，看它在每个维度上做了什么选择。

**理解一个技术领域（前端/后端/系统架构）**：从对应的 `domains/` 文档开始——它们从零演进，逐层解释 WHY。

**了解生态现状与学习建议**：读 `domains/ecosystem-state.md`——基于数据的语言趋势、框架格局和 2026 年学习建议。

**寻找优质开源项目学习代码**：读 `domains/reference-projects.md`——按技术栈和难度分级的全栈项目对比。

## 适合谁

- 已有编程基础，需要快速切入新语言的开发者
- 使用 Neovim/Helix/Emacs 等轻量编辑器，需自行搭建开发环境
- 希望理解技术生态全貌而非停留在语法层面的学习者

## 许可

CC0 — 自由使用，欢迎贡献和衍生。
