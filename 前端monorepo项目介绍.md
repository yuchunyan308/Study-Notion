# 前端 Monorepo 项目详解

## 什么是 Monorepo？

Monorepo（单一仓库）是一种将**多个项目/包**放在同一个 Git 仓库中管理的架构策略，与之对应的是 Polyrepo（多仓库，每个项目独立一个 repo）。

```
# Polyrepo（传统方式）
repo-app-web/
repo-app-mobile/
repo-ui-components/
repo-utils/

# Monorepo（单一仓库）
my-monorepo/
├── apps/
│   ├── web/
│   └── mobile/
└── packages/
    ├── ui-components/
    └── utils/
```

---

## 核心目录结构

```
my-monorepo/
├── apps/                        # 应用层
│   ├── web/                     # React / Next.js 主站
│   ├── admin/                   # 管理后台
│   ├── mobile/                  # React Native 应用
│   └── docs/                    # 文档站点 (VitePress / Docusaurus)
│
├── packages/                    # 共享包层
│   ├── ui/                      # 组件库
│   │   ├── src/
│   │   │   ├── Button/
│   │   │   ├── Modal/
│   │   │   └── index.ts
│   │   └── package.json
│   ├── utils/                   # 工具函数
│   ├── hooks/                   # 共享 React Hooks
│   ├── config/                  # 共享配置（ESLint、TSConfig 等）
│   │   ├── eslint-config/
│   │   └── tsconfig/
│   └── types/                   # 共享 TypeScript 类型
│
├── tools/                       # 构建脚本、CLI 工具
├── package.json                 # 根 package.json（workspace 配置）
├── pnpm-workspace.yaml          # pnpm workspace 配置
├── turbo.json                   # Turborepo 配置
└── tsconfig.base.json           # 根 TypeScript 配置
```

---

## 主流工具链

### 1. 包管理器（Workspace 支持）

| 工具 | 配置文件 | 特点 |
|------|---------|------|
| **pnpm** ⭐ | `pnpm-workspace.yaml` | 最流行，磁盘效率高，速度快 |
| **npm** | `package.json workspaces` | 原生支持，无需额外安装 |
| **Yarn Berry** | `.yarnrc.yml` | PnP 模式，零安装 |

```yaml
# pnpm-workspace.yaml
packages:
  - 'apps/*'
  - 'packages/*'
  - 'tools/*'
```

---

### 2. 构建编排工具（Task Runner）

#### Turborepo ⭐（最主流）
```json
// turbo.json
{
  "$schema": "https://turbo.build/schema.json",
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],   // 依赖包先 build
      "outputs": ["dist/**", ".next/**"]
    },
    "lint": {
      "outputs": []
    },
    "test": {
      "dependsOn": ["build"],
      "outputs": ["coverage/**"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    }
  }
}
```

#### Nx（企业级）
```json
// nx.json
{
  "affected": { "defaultBase": "main" },
  "tasksRunnerOptions": {
    "default": {
      "runner": "nx/tasks-runners/default",
      "options": { "cacheableOperations": ["build", "test", "lint"] }
    }
  }
}
```

---

### 3. 构建工具

| 工具 | 适用场景 |
|------|---------|
| **Vite** | 应用构建、组件库开发 |
| **tsup** | 纯 TypeScript 库打包（极简） |
| **Rollup** | 精细控制的库打包 |
| **esbuild** | 超高速构建，常作为底层 |
| **Webpack** | 遗留项目兼容 |

---

## 包之间的依赖关系

```json
// apps/web/package.json
{
  "name": "@myorg/web",
  "dependencies": {
    "@myorg/ui": "workspace:*",       // 引用本地 ui 包
    "@myorg/utils": "workspace:*",    // 引用本地 utils 包
    "@myorg/types": "workspace:*"
  }
}
```

```ts
// apps/web/src/App.tsx
import { Button, Modal } from '@myorg/ui'
import { formatDate } from '@myorg/utils'
import type { User } from '@myorg/types'
```

依赖图示：
```
web ──► ui ──► utils
  │              ▲
  └──────────────┘
admin ──► ui
      └──► utils
```

---

## 关键特性详解

### ① 增量构建 & 缓存

Turborepo 会对每个任务的输入（源文件 hash）做缓存，**只重新构建变更的包**：

```bash
# 第一次构建（全量）
turbo run build  →  构建 12 个包，耗时 45s

# 修改 ui 包后再构建
turbo run build  →  仅重建 ui + 依赖它的 web/admin，耗时 8s
                    其余包命中缓存 ✓
```

### ② 远程缓存（Remote Cache）

```bash
# 团队成员 A 构建后，成员 B 直接复用缓存（CI 同理）
turbo run build --team="myorg" --token="xxx"
# >>> FULL TURBO（100% 缓存命中）
```

### ③ 并行执行

```
build 任务并行图：
types ──► utils ──► ui ──┐
                          ├──► web   (并行构建)
                          └──► admin (并行构建)
```

---

## TypeScript 配置共享

```json
// packages/config/tsconfig/base.json
{
  "compilerOptions": {
    "target": "ES2020",
    "moduleResolution": "bundler",
    "strict": true,
    "paths": {}
  }
}

// apps/web/tsconfig.json
{
  "extends": "@myorg/tsconfig/base.json",
  "compilerOptions": {
    "jsx": "react-jsx",
    "outDir": "./dist"
  },
  "include": ["src"]
}
```

---

## ESLint 配置共享

```js
// packages/config/eslint-config/index.js
module.exports = {
  extends: ['eslint:recommended', 'plugin:react/recommended'],
  rules: { 'no-console': 'warn' }
}

// apps/web/.eslintrc.js
module.exports = {
  extends: ['@myorg/eslint-config'],
  rules: { 'no-console': 'error' }  // 覆盖特定规则
}
```

---

## 常用命令模式

```bash
# 全局安装依赖
pnpm add -w lodash

# 给特定包安装依赖
pnpm add axios --filter @myorg/web

# 运行所有包的 dev
turbo run dev

# 只运行受影响的测试（基于 git diff）
turbo run test --filter=...[origin/main]

# 运行特定包
turbo run build --filter=@myorg/ui
turbo run build --filter=@myorg/web...  # 包含所有依赖
```

---

## Monorepo vs Polyrepo 对比

| 维度 | Monorepo | Polyrepo |
|------|---------|---------|
| **代码复用** | ✅ 直接引用，无需发布 | ❌ 需发布 npm 包 |
| **原子提交** | ✅ 跨包改动一次 commit | ❌ 多 repo 难以协调 |
| **统一工具链** | ✅ 一套 lint/test/CI | ❌ 各自维护 |
| **构建速度** | ✅ 增量缓存 | ✅ 互不干扰 |
| **权限控制** | ❌ 较难细粒度控制 | ✅ 每 repo 独立权限 |
| **仓库体积** | ❌ 随时间增大 | ✅ 各自独立 |
| **新人上手** | ✅ 一个 repo 克隆即用 | ❌ 需克隆多个 repo |

---

## 推荐技术选型（2025 最佳实践）

```
包管理：  pnpm workspaces
编排：    Turborepo
应用框架：Next.js / Vite + React
库打包：  tsup
类型：    TypeScript（共享 tsconfig）
测试：    Vitest（支持 workspace）
CI：      GitHub Actions + Turbo 远程缓存
```

---

## 典型 CI/CD 配置

```yaml
# .github/workflows/ci.yml
name: CI
on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with: { fetch-depth: 2 }       # 用于 affected 分析

      - uses: pnpm/action-setup@v3
      - run: pnpm install --frozen-lockfile

      - name: Build & Test（增量）
        run: turbo run build test lint
        env:
          TURBO_TOKEN: ${{ secrets.TURBO_TOKEN }}  # 远程缓存
          TURBO_TEAM: ${{ vars.TURBO_TEAM }}
```

---

Monorepo 的核心价值在于：**统一协作、消除重复、加速迭代**。对于中大型团队或需要共享大量代码的项目，是目前最主流的工程化选择。
