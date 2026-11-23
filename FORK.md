# Fork 说明

这是 [pingdotgg/t3code](https://github.com/pingdotgg/t3code) 的个人 fork，用来做自用的定制功能，主要面向 macOS 桌面端。上游的 [AGENTS.md](AGENTS.md)、[CONTRIBUTING.md](CONTRIBUTING.md) 和 `docs/internals/` 全部继续有效，本文件只记录 fork 自己的约定。

## 远程与分支

| 名称          | 用途                                                                  |
| ------------- | --------------------------------------------------------------------- |
| `upstream`    | 官方仓库，只拉取，不推送                                              |
| `origin`      | 自己的 fork                                                           |
| `main`        | 只镜像 `upstream/main`，永远不在上面提交                              |
| `custom`      | 定制集成分支 = `upstream/main` + 定制提交（每个功能尽量压成一个提交） |
| `feat/<name>` | 从 `custom` 切出的单个定制，完成后 squash 合回 `custom`               |
| `fix/<name>`  | 从 `upstream/main` 切出的、准备回馈上游的 bug 修复                    |

本地已开启 `git rerere`，同一处冲突解决过一次后，下次 rebase 会自动复用。

## 迭代流程

1. 开始新定制：按 `fork-feature` skill 走（[SKILL.md](.agents/skills/fork-feature/SKILL.md)）。
2. 同步上游：大约每周一次，按 `fork-sync` skill 走（[SKILL.md](.agents/skills/fork-sync/SKILL.md)）。拖得越久冲突越多。
3. 每个定制合进 `custom` 时，在下面的「定制清单」登记一行。rebase 解冲突时以这张表为准，判断哪些改动是我们的。

## 让定制在 rebase 中活下来

- **新增优于修改**：优先新建文件、新组件、新 service，在已有文件里只留一个小的接入点。被改动的上游文件越少，冲突越少。
- **改在边界**：上游的约定是"复杂度放在 adapter 边界，编排保持纯粹，UI 保持简单"，定制也照这个来。不改 `Orchestrator`、事件存储这类核心路径。
- **一个功能一个提交**：`custom` 上每个定制尽量只占一个提交，rebase 时冲突能按功能逐个处理，必要时整功能 drop。
- **不做顺手的重构**：格式化、重命名、顺手清理都会放大和上游的 diff。

## 高风险区（改之前先和开发者确认）

- **数据库迁移**：`apps/server/src/persistence/Migrations/` 按顺序编号（目前到 110 左右），在 `Migrations.ts` 里注册。Migrator 只会执行比"已记录的最大编号"更大的迁移。如果我们加了 111，上游之后也发一个 111，上游那一条在我们的库上就永远不会执行，数据库结构会悄悄坏掉。默认不加迁移；确实需要持久化时，优先用不依赖迁移的方式（比如独立的设置项或文件）。
- **`packages/contracts`**：线上协议，server、web、mobile、desktop 都依赖它，上游改动频繁，冲突概率最高。
- **上游的 `AGENTS.md`**：只在文件末尾的「Fork 层」小节里写 fork 规则，不改上面的上游内容。
- **桌面端自动更新**（`apps/desktop/src/updates/`）：自己打包的 App 默认会跟官方的更新源走，可能被官方版本覆盖。等到要替换官方 App 时，再单独处理。

## 运行与数据安全

- 日常仍然用官方的 `T3 Code (Beta).app`。定制版用 `vp run dev:desktop` 跑开发版。
- 主 checkout 的开发状态在 `~/.t3/dev`，linked worktree 的在 `<worktree>/.t3/userdata`。**绝不能**让开发服务器指向 `~/.t3/userdata`，那是官方 App 正在使用的真实数据库。
- 免费的 `Replay (dev)` provider 可以用来测试 UI，不消耗模型额度。

## 回馈上游

- 只在开发者明确要求时才开 PR 或 issue。
- 修复从 `upstream/main` 切 `fix/<name>`，不能带任何定制代码。
- 遵守 [CONTRIBUTING.md](CONTRIBUTING.md)：只收 bug 修复，一个 PR 只解决一个问题，每人最多同时开 3 个 PR。作者声明由 Gemini 模型（Gemini 4 Neon 除外）或 Muse Wonder 写的代码会被拒。

## 工具

- 已装：Vite+（`vp`，自带固定版本的 Node）、Rust（`cargo`，用于 resource-monitor 和 DMG 打包）、GitHub CLI（`gh`）。
- MCP：暂不额外安装。Effect 的参考资料用仓库自带、与项目版本一致的 `.repos/effect`（`LLMS.md` + 源码）；GitHub 操作用 `gh`；UI 验证用 T3 自带的 preview 工具和 `test-t3-app` skill。
- Skills：上游自带的在 `.agents/skills/`（`.claude/skills` 是指向它的软链接），fork 自己加的是 `fork-feature` 和 `fork-sync`。

## 定制清单

| 功能     | 提交 | 改动的上游文件 | 新增文件 | 备注 |
| -------- | ---- | -------------- | -------- | ---- |
| （暂无） |      |                |          |      |

## 同步记录

| 日期       | 同步到的 upstream 提交 | 备注        |
| ---------- | ---------------------- | ----------- |
| 2026-11-23 | `d63b9c4c76`           | fork 初始化 |
