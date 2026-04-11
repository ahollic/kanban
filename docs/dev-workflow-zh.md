# Kanban 本地开发指南

## 前置条件

- Node.js 20+
- npm 10+

## 安装依赖

```bash
# 同时安装根项目和 web-ui 的依赖
npm run install:all
```

## 开发模式

### 方式一：仅运行后端（不含前端热更新）

```bash
npm run dev
```

- 启动后端服务，运行在 `http://127.0.0.1:3484`
- 后端使用 `tsx watch` 监听 `src/` 下的文件变更，**修改后端代码会自动重启**
- 前端页面从 `web-ui/dist/` 读取**已构建的静态文件**，**不支持热更新**
- 首次使用前需要先构建一次前端：`npm run web:build`

> 适用于：只改后端代码，不涉及前端修改的场景。

### 方式二：前后端同时开发（推荐）

需要开两个终端：

**终端 1 — 后端服务：**

```bash
npm run dev
```

**终端 2 — 前端 Vite 开发服务器：**

```bash
npm run web:dev
```

- Vite 开发服务器运行在 `http://127.0.0.1:4173`
- 所有 `/api/*` 请求自动代理到后端 `http://127.0.0.1:3484`
- **修改前端代码会即时热更新（HMR）**

> 浏览器访问 `http://127.0.0.1:4173`（Vite 地址），不要访问后端地址。

## 全局链接（`npm link`）

`npm link` 将当前项目注册为全局命令，让你可以在**任何目录**下直接使用 `kanban` 命令。

### 安装全局链接

```bash
npm run link
```

这个命令实际执行的是 `npm run build && npm link`，即先完整构建，再创建全局符号链接。

### 验证

```bash
which kanban
kanban --version
```

### 使用

```bash
cd /path/to/any/project
kanban
```

### ⚠️ 重要：修改代码后需要重新构建

`npm link` 链接的是 `dist/` 目录下的**构建产物**，不是源码。因此：

| 改了什么 | 需要做什么 |
|---|---|
| 后端代码（`src/`） | 重新 `npm run build` |
| 前端代码（`web-ui/`） | 重新 `npm run build`（包含前端构建 + 拷贝到 `dist/web-ui/`） |
| 两者都改了 | 重新 `npm run build` |

每次 build 后，`dist/` 内容更新，全局 `kanban` 命令就会使用最新代码。

### 在多个 worktree 之间切换

如果你有多个 git worktree，需要从目标 worktree 重新执行 `npm run link`，让全局 `kanban` 指向正确的 `dist/cli.js`。

### 移除全局链接

```bash
npm run unlink
```

## 构建与运行打包后的 CLI

```bash
# 完整构建
npm run build

# 直接运行
node dist/cli.js

# 指定端口
node dist/cli.js --port 3484

# 自动选取可用端口（从 3484 开始）
node dist/cli.js --port auto
```

此模式下前端从 `dist/web-ui/` 读取，不支持热更新。

## 各命令速查

| 命令 | 说明 |
|---|---|
| `npm run install:all` | 安装根项目 + web-ui 依赖 |
| `npm run dev` | 后端 watch 模式（后端热重启，前端静态） |
| `npm run web:dev` | 前端 Vite 开发服务器（前端热更新） |
| `npm run web:build` | 仅构建前端 |
| `npm run build` | 完整构建（前端 + 后端） |
| `npm run link` | 构建 + 注册全局 `kanban` 命令 |
| `npm run unlink` | 移除全局 `kanban` 命令 |
| `npm run typecheck` | 后端类型检查 |
| `npm run web:typecheck` | 前端类型检查 |
| `npm run test` | 后端测试 |
| `npm run web:test` | 前端测试 |
| `npm run check` | lint + typecheck + 测试 |
| `npm run lint` | 代码检查 |
| `npm run format` | 格式化代码 |

## 常见场景

### 场景 1：日常全栈开发

```bash
# 终端 1
npm run dev

# 终端 2
npm run web:dev

# 浏览器访问 http://127.0.0.1:4173
```

前端改动即时生效，后端改动自动重启。

### 场景 2：只改后端

```bash
npm run dev
# 浏览器访问 http://127.0.0.1:3484
# 前提：已构建过前端（npm run web:build）
```

### 场景 3：在其他项目中测试

```bash
# 首次设置
npm run link

# 之后每次改完代码
npm run build

# 在任意目录使用
cd /path/to/your/project
kanban
```

### 场景 4：提交 PR 前

```bash
npm run check      # lint + typecheck + 测试
npm run build      # 确保构建无报错
```

