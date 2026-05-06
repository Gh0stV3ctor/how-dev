# 05 — 项目结构约定全景

## 为什么项目结构有约定

当你打开一个陌生仓库，以下问题决定了你多快能理解它：

- 源码在哪？（`src/`？根目录？）
- 测试在哪？（`test/`？`__tests__/`？和源码放一起？）
- 构建入口是什么？（`Makefile`？`CMakeLists.txt`？`pyproject.toml`？）
- 配置怎么组织？（`config/`？`.env`？`.config/`？）

项目结构不是随意的——每种社区约定都是该语言编译模型、工具链假设和工程哲学的自然产物。

## 为什么不同语言有不同的结构

根本原因：**编译模型和工具链决定了结构的可能性**。

- **C/C++**：独立编译 `.c` 文件 + 头文件 → 必须区分"公开声明"（`include/`）和"实现"（`src/`）
- **Python**：`import` 按目录结构搜索 → 目录结构直接影响代码的导入路径
- **Go**：包路径即导入路径 → 目录层级影响 API 的组织
- **Rust**：模块系统由 `mod` 声明驱动 → 文件和目录是模块树的映射
- **JavaScript**：历史上无标准 → 社区百花齐放，现代收敛于有限的几种模式

## 通用模式

### 源码组织模式

| 模式 | 结构 | 适用 | 语言 |
|------|------|------|------|
| **src layout** | `src/pkgname/` + `tests/` | 库项目 | Python（现代推荐）、C/C++（`src/`+`include/`） |
| **flat layout** | 包目录放在根目录 | 简单项目 | Python（旧式）、Go（根目录即包） |
| **cmd/internal pattern** | `cmd/`（入口）+ `internal/`（内部包）+ `pkg/`（公共包） | 多二进制项目 | Go 事实标准 |
| **src/ + include/ 分离** | 源文件和头文件分离 | 库项目 | C/C++ |
| **workspace** | 多个 crate 共享 target 目录 | 大型项目 | Rust |
| **monorepo with packages/** | `packages/lib-a`、`packages/app-b` | 多包项目 | JavaScript（npm workspaces/turborepo） |

### 测试放置策略

| 策略 | 结构 | 语言 |
|------|------|------|
| **同目录测试** | `src/foo_test.go` 和 `src/foo.go` 同目录 | Go（`_test.go` 后缀） |
| **内联测试模块** | `#[cfg(test)] mod tests { ... }` 写在源码文件底部 | Rust |
| **独立 tests/ 目录** | `tests/test_foo.py` | Python、C/C++、JS（集成测试） |
| **__tests__ 约定** | `__tests__/` 和源码同级的测试目录 | JS（Jest 默认） |
| **test/ 子目录** | `test/test_parser.cpp` | C/C++ |

### 配置文件放置

| 放置位置 | 示例 | 语言/工具 |
|----------|------|----------|
| **根目录** | `pyproject.toml`、`go.mod`、`Cargo.toml`、`package.json` | 几乎所有现代语言 |
| **隐藏目录** | `.github/`、`.circleci/` | CI 配置 |
| **config/ 目录** | `config/settings.toml` | 应用自身配置 |
| **环境变量 + .env** | `.env`（不提交）、`.env.example`（提交） | 跨语言通用 |

## 各语言标准结构

### C/C++ 项目

```
project/
├── src/               # .c/.cpp 源文件
├── include/           # 公开头文件（对外 API）
│   └── project/       #   安装到 /usr/include/project/
├── internal/           # 内部头文件（不对外）
├── test/
├── third_party/       # vendored 依赖
├── CMakeLists.txt     # 或 Makefile / meson.build
├── .clang-format
└── .clang-tidy
```

**关键点**：`include/` vs `internal/` 的分离是 C/C++ 独有的——因为头文件需要被**安装到系统**（`-I/usr/include/project`），必须明确哪些是公开 API。

### Python 项目

```
project/
├── pyproject.toml
├── src/                         # 源码（src layout，推荐）
│   └── mypackage/
│       ├── __init__.py
│       ├── core.py
│       └── utils/
│           └── __init__.py
├── tests/
│   ├── test_core.py
│   └── conftest.py              # pytest fixtures
├── docs/
└── .venv/                       # 虚拟环境（不提交）
```

**Python 特有**：
- `__init__.py` 标记目录为包（Python 3.3+ 可省略但极不推荐）
- **src layout vs flat layout**：`src/mypackage/` 强制先安装包才能导入，避免意外导入源码目录。现代项目推荐 src layout
- **没有单独的 include**：Python 没有头文件概念

### Go 项目

```
project/
├── go.mod
├── go.sum
├── main.go                       # 单二进制入口
├── cmd/                          # 多二进制入口（可选）
│   ├── server/main.go
│   └── worker/main.go
├── internal/                     # 编译器强制不被外部导入
│   └── storage/
├── pkg/                          # 公共库（有争议）
├── api/                          # proto / OpenAPI 定义
├── test/                         # 集成测试数据
└── scripts/
```

**Go 特有**：
- **`internal/` 是编译器级别的封装边界**——外部模块无法 import
- **不使用 `src/` 目录**——包在根目录下的子目录中
- **包路径和目录路径直接对应**：`import "github.com/user/project/internal/storage"`

### Rust 项目

```
project/
├── Cargo.toml
├── Cargo.lock
├── src/
│   ├── main.rs                  # binary crate 入口
│   ├── lib.rs                   # library crate 入口
│   ├── error.rs
│   └── utils.rs
├── tests/                       # 集成测试
│   └── integration_test.rs
├── examples/                    # 示例（cargo run --example）
├── benches/                     # 基准测试
├── build.rs                     # 构建脚本（C 绑定等场景）
└── target/                      # 构建产物（不提交）
```

**Rust 特有**：
- `lib.rs` vs `main.rs` 决定 crate 类型
- 模块体系由 `mod` 声明驱动，不是文件系统驱动
- workspace 模式（多 crate）：`[workspace] members = ["crates/*"]`

### JavaScript / TypeScript 项目

```
project/
├── package.json
├── tsconfig.json                 # TypeScript 配置
├── vite.config.ts                # 或 webpack.config.js、next.config.js 等
├── index.html                    # 前端入口
├── src/
│   ├── main.ts                   # 应用入口
│   ├── components/
│   ├── utils/
│   └── types/
├── public/                       # 静态资源（直接复制到构建产物）
├── tests/
│   └── *.test.ts
├── node_modules/                 # 依赖（不提交）
└── dist/                         # 构建产物（不提交）
```

**JS/TS 特有**：
- 没有"一个标准"——Vite 项目、Next.js 项目、npm 库各有不同约定
- Next.js 使用文件系统路由（`app/page.tsx` → URL `/`）
- `public/` vs `static/` 在不同框架中命名不同
- 测试文件通常用 `.test.ts` 或 `.spec.ts` 后缀

## 单仓库（Monorepo）模式

当项目包含多个可独立发布的包时，需要更高层的组织：

```
monorepo/
├── package.json                  # workspace 根（npm workspaces）
├── pnpm-workspace.yaml           # 或 pnpm workspace
├── turbo.json                    # Turborepo 配置
├── packages/
│   ├── shared-utils/
│   │   ├── package.json
│   │   └── src/
│   ├── web-app/
│   │   ├── package.json
│   │   └── src/
│   └── api-server/
│       ├── package.json
│       └── src/
└── .github/
```

各语言的 monorepo 方案：

| 语言 | 方案 |
|------|------|
| JavaScript | npm workspaces / pnpm workspaces + Turborepo / Nx / Lerna |
| Rust | Cargo workspace（`[workspace]` in `Cargo.toml`） |
| Go | 单模块多 `cmd/` 目录 / Go workspace（`go.work`，Go 1.18+） |
| Python | `pyproject.toml` 的 `[tool.uv.workspace]` / Pantsbuild / Bazel |

## 关键规律

1. **头文件语言必须有 "include" 和 "src" 分离**（C/C++）——因为头文件是对外合同
2. **有 import 机制的语言结构中目录即命名空间**（Python、Go）——不能随意挪文件
3. **模块系统决定文件组织方式**（Rust `mod` vs Go 目录 vs Python `__init__.py`）
4. **单二进制项目最简单**（Go、Rust 的 `main.go`/`main.rs`），多入口项��需要 `cmd/` 目录
5. **构建产物目录**（`build/`、`target/`、`dist/`、`node_modules/`）**永远不应提交**
6. **测试放置策略**取决于测试框架的约定和语言是否支持内联测试模块
