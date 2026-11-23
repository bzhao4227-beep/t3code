---
name: fork-feature
description: Plan, build, verify, and land a personal customization on this T3 Code fork's custom branch with a minimal, rebase-friendly diff. Use when the developer asks for a new custom feature or behavior change in their fork (not an upstream bug-fix PR).
---

# 新增一个定制

## 1. 先查再做

- 弄清楚需求。如果有明显更简单的做法，或者其实只需要改设置，先告诉开发者。
- 查上游是否已经有了或正在做：`git log upstream/main --oneline -i --grep=<关键词>`，`gh search prs --repo pingdotgg/t3code <关键词>`，`gh search issues --repo pingdotgg/t3code <关键词>`。上游很快会做的东西，自己再做一遍只会制造冲突。
- 按 [AGENTS.md](../../../AGENTS.md) 的「Internal docs by area」表，先读对应领域的 `docs/internals/` 文档。

## 2. 设计成"最小 diff"

动手前列出准备改的上游文件和准备新增的文件，规则见 [FORK.md](../../../FORK.md)「让定制在 rebase 中活下来」。涉及数据库迁移、`packages/contracts`、核心编排，或者要改的上游文件超过五六个时，先把方案给开发者确认。

覆盖面：定制默认只需要支持 macOS 桌面端（即 server + Web UI）。移动端、Windows/Linux 可以明确不支持，但不能被弄坏，类型检查和测试仍然要通过。新增的界面文案按上游 i18n 流程处理（`vp run i18n:generate` / `vp run i18n:check`）。

## 3. 实现

```sh
git switch custom
git switch -c feat/<name>
```

遵守上游 AGENTS.md 的 Taste 规则：Effect service + `Effect.fn`、不用 `any`、默认不写注释、React effect 要写明理由。迭代时用 `vp test run <files>` 和 `vp run --filter <pkg> typecheck`。

## 4. 提交并验证

在功能分支更新 FORK.md 的「定制清单」。按路径暂存本次实现、测试和清单，检查 `git diff --cached` 后用英文 conventional commit 提交。确认 `git status --short` 输出为空，再验证已提交的功能。

```sh
vp run check:changed custom      # 只检查这个分支相对 custom 的改动
```

用户能看到的改动，要在真实客户端里走一遍：`vp run dev:desktop`，或者用 `test-t3-app` skill。默认用 `Replay (dev)` provider。

## 5. 合入 custom

```sh
git switch custom
git merge --squash feat/<name>
git commit -m "feat(fork): <plain-language summary>"
```

用 `git diff --stat HEAD^ HEAD` 核对合入的实现和清单，再运行 `vp run check:changed main` 检查集成结果。推送到 `origin` 需要开发者明确同意。
