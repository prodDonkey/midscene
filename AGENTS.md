# AGENTS.md

本文件是本仓库中面向 coding agent 的规范来源。`CLAUDE.md`
应当指向这里，而不是重复维护一份规则。

## 默认工作流

- 仅使用 `pnpm`。本工作区要求 Node `>=18.19.0`，pnpm `>=9.3.0`。
- 在本地开发前先阅读 `CONTRIBUTING.md`。开发/构建流程、应用本地 dev server
  启动方式，以及报告重建的排障说明都维护在那里，以避免重复。
- 在创建 commit 或更新 PR 之前，在仓库根目录执行 `pnpm run lint`。
- 对于代码变更，针对每个受影响项目运行最小必要的 Nx target，不要默认执行整
  个 monorepo 的全量校验。
- AI 测试依赖部分环境变量，例如需要设置 `MIDSCENE_MODEL_BASE_URL`。

## 真正重要的变更规则

- 当行为发生变化时，新增或更新测试。优先从最近的单元测试套件开始；只有在变
  更确实依赖模型行为或浏览器/设备集成时，才使用 AI 测试或 e2e。
- 不要手动编辑 `dist/` 或 `apps/site/doc_build/` 下的生成产物。
- 当修改共享 package 或导出的入口点时，结束前要为受影响项目执行一次聚焦构建。

## Commit 与 PR 规则

- Commit 必须遵循 Conventional Commits，并且 scope 是必填项。
- Scope 取值来自 `apps/` 和 `packages/` 下的目录名，以及
  `commitlint.config.js` 中定义的共享 scope，例如 `workflow`、`llm`、
  `playwright`、`puppeteer` 和 `bridge`。
- 重要例外：修改 `packages/web-integration` 下的内容时，commit scope 要使
  用 `web-integration`，即使其发布后的包名是 `@midscene/web`。
- 重要例外：修改 `apps/site` 时，commit scope 要使用 `site`，即使其 Nx
  project 名称是 `doc`。
- 在 PR 摘要中，列出你实际执行过的校验命令。

## 文档与国际化

- 默认把面向用户的文档视为双语内容。
- 如果你修改了 `README.md`，需要在同一次变更中同步更新 `README.zh.md`。
- 如果你修改了 `apps/site/docs/en/**`，检查并更新
  `apps/site/docs/zh/**` 下对应的文件；反向修改时也同样适用。
- 英文和中文目录树并不是完全镜像的。如果对应文件不存在，需要决定是补上，
  还是在最终总结中说明这是有意保留的差异。
- 修改站点文案前，先阅读 `apps/site/agents.md` 中的术语规则。该文件已经说
  明了一些翻译约束，例如在中文中某些情况下应保留 `API Key` 和 `Agent` 原文。

## 分支与 Remote 管理

本地仓库已配置双 remote：

- `origin` → 个人 fork：`git@github.com:prodDonkey/midscene.git`（推送自己的改动到这里）
- `upstream` → 上游原仓库：`git@github.com:web-infra-dev/midscene.git`（拉取上游新特性从这里）

**日常同步上游：**
```bash
git fetch upstream
git checkout main
git rebase upstream/main
git push origin main
```

**开发新改动（每个任务新建子分支，完成后合并回 feature/yhl）：**
```bash
git checkout feature/yhl
git checkout -b feature/yhl-<任务简述>
# ... 开发 ...
git checkout feature/yhl
git merge --no-ff feature/yhl-<任务简述>
git branch -d feature/yhl-<任务简述>
```

**将上游更新同步到开发分支：**
```bash
git checkout main
git rebase upstream/main
git checkout feature/yhl
git rebase main
```

原则：`main` 只跟踪上游，`feature/yhl` 是个人主开发分支，每个具体任务在其子分支上完成后合并回来，用 rebase 而非 merge 跟踪上游。

## Commit 语言规范

**所有 commit message 必须使用中文**，格式仍遵循 Conventional Commits（scope 必填）：

```
类型(scope): 中文描述

# 示例
feat(core): 新增多模型并发请求支持
fix(web-integration): 修复页面截图偶发空白问题
docs(site): 更新快速开始文档的安装步骤
refactor(llm): 提取公共的 token 计数工具函数
```

类型对照：`feat` 新功能、`fix` 修复、`docs` 文档、`refactor` 重构、`test` 测试、`chore` 杂项。

## 校验建议

- 仅文档变更：通常执行 `pnpm run lint` 即可。
- 单 package 代码变更：执行 `pnpm run lint`，再执行最小相关的
  `npx nx test <project>`；如果涉及导出或构建接线变更，再执行
  `npx nx build <project>`。
- 跨 package 的运行时或构建系统变更：执行 `pnpm run lint`，并明确说明是否
  还有更大范围的校验尚未完成。
