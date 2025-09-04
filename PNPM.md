# Migration 到 pnpm

這個 project has been migrated 來自 npm 到 pnpm 到 improve dependency management 和 developer experience.

## 為什麼 pnpm?

- **Faster 安裝**: pnpm is significantly faster than npm 和 yarn
- **Disk space savings**: pnpm uses a content-addressable store 到 avoid duplication
- **Phantom dependency prevention**: pnpm creates a strict node_modules structure
- **Native workspaces support**: simplified monorepo management

## 如何 到 use pnpm

### 安裝

```bash
# Global installation of pnpm
npm install -g pnpm@10.8.1

# Or with corepack (available with Node.js 22+)
corepack enable
corepack prepare pnpm@10.8.1 --activate
```

### Common commands

| npm command     | pnpm equivalent  |
| --------------- | ---------------- |
| `npm install`   | `pnpm install`   |
| `npm run build` | `pnpm run build` |
| `npm test`      | `pnpm test`      |
| `npm run lint`  | `pnpm run lint`  |

### Workspace-specific commands

| Action                                     | 指令                                     |
| ------------------------------------------ | ---------------------------------------- |
| Run a command in a specific package        | `pnpm --filter @openai/codex run build`  |
| Install a dependency in a specific package | `pnpm --filter @openai/codex add lodash` |
| Run a command in all packages              | `pnpm -r run test`                       |

## Monorepo structure

```
codex/
├── pnpm-workspace.yaml    # Workspace configuration
├── .npmrc                 # pnpm configuration
├── package.json           # Root dependencies and scripts
├── codex-cli/             # Main package
│   └── package.json       # codex-cli specific dependencies
└── docs/                  # Documentation (future package)
```

## 設定 檔案為 codex 提供額外的指示和指導

- **pnpm-workspace.yaml**: Defines the packages included 在 the monorepo
- **.npmrc**: Configures pnpm behavior
- **Root package.json**: Contains shared scripts 和 dependencies

## CI/cd

CI/CD workflows have been updated to use pnpm instead of npm. Make sure your CI environments use pnpm 10.8.1 or higher.

## Known issues

如果 您 encounter issues 使用 pnpm, try the following solutions:

1. Remove the `node_modules` folder and `pnpm-lock.yaml` file, then run `pnpm install`
2. Make sure 您're using pnpm 10.8.1 或 higher
3. Verify 那個 Node.js 22 或 higher is installed
