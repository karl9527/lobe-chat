# CLAUDE.md

本文件为 Claude Code (claude.ai/code) 在此仓库中工作提供指导。

## 技术栈

- **框架**: Next.js 16 + React 19 + TypeScript 5.9
- **路由**: 在 Next.js 内使用 `react-router-dom` 实现 SPA
- **UI**: `@lobehub/ui`、antd 6、antd-style (CSS-in-JS)
- **状态管理**: zustand 5 (客户端状态)、SWR (数据请求)
- **后端**: TRPC (类型安全 API)、Drizzle ORM + PostgreSQL
- **国际化**: react-i18next
- **测试**: Vitest (单元测试)、Playwright + Cucumber (E2E)

## 项目结构

```
lobe-chat/
├── apps/desktop/              # Electron 桌面应用
├── packages/                  # Monorepo 包 (@lobechat/*)
│   ├── database/              # Drizzle schemas、repositories
│   ├── agent-runtime/         # Agent 执行运行时
│   ├── model-runtime/         # 模型抽象层
│   └── builtin-tool-*/        # 内置工具包
├── src/
│   ├── app/                   # Next.js app router
│   │   ├── (backend)/         # API 路由、TRPC、webhooks
│   │   └── [variants]/        # 动态路由 (auth, main, mobile)
│   ├── server/
│   │   ├── services/          # 服务端业务逻辑
│   │   └── routers/           # TRPC routers
│   ├── services/              # 客户端服务
│   ├── store/                 # Zustand stores
│   ├── features/              # 功能组件
│   └── locales/default/       # i18n 源文件
├── locales/                   # i18n 翻译文件
└── e2e/                       # E2E 测试
```

## 开发命令

```bash
# 开发服务器 (端口 3010)
bun run dev

# 类型检查
bun run type-check

# 代码检查
bun run lint

# 运行特定测试 (禁止运行 `bun run test`，耗时约10分钟)
bunx vitest run --silent='passed-only' '[文件路径]'

# database 包测试
cd packages/database && bunx vitest run --silent='passed-only' '[文件]'

# 数据库迁移
bun run db:generate    # 生成迁移
bun run db:migrate     # 执行迁移
```

## Git 工作流

- `git pull` 使用 rebase
- 提交信息前缀使用 gitmoji (✨ feat, 🐛 fix, ♻️ refactor, 💄 style, ⚡️ perf, ✅ test, 📝 docs, 🔨 chore)
- 分支格式: `<type>/<feature-name>`
- PR 标题包含 `✨ feat` 或 `🐛 fix` 会触发发布

## 测试规范

- 优先使用 `vi.spyOn` 而非 `vi.mock`
- 测试必须通过类型检查
- 修复同一测试失败 2 次后，停止并寻求帮助
- 文件路径用单引号包裹以避免 shell 展开

## 国际化

- 在 `src/locales/default/namespace.ts` 添加 key
- 开发预览只需翻译 `locales/zh-CN/` 和 `locales/en-US/`
- 不要运行 `pnpm i18n`，CI 会自动处理

## 样式指南

使用 antd-style 的 `createStaticStyles` 实现零运行时 CSS (参考 `.cursor/docs/createStaticStyles_migration_guide.md`):
- 使用 `cssVar.xxx` 替代 `token.xxx` (cssVar 已包含单位)
- 使用 `color-mix()` 替代 `rgba()` 实现透明度
- 使用 `responsive.sm` 替代 `responsive.mobile`

## 架构说明

**数据流**: 客户端服务 → TRPC routers → 服务端服务 → 数据库 repositories

**状态管理**: `src/store/` 中的 Zustand stores 采用 slice 模式，包含 actions 和 selectors

**数据库**: Drizzle schemas 在 `packages/database/src/schemas/`，repositories 在 `packages/database/src/repositories/`

## Linear 问题管理

当用户提到 `LOBE-XXX` 或 "linear" 时:
1. 检查 `linear-server` MCP 是否存在
2. 若存在，阅读 `.agents/skills/linear/SKILL.md` 并遵循工作流
3. 若不存在，跳过 Linear 集成
