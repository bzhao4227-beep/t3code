---
name: fork-sync
description: Sync this personal T3 Code fork with upstream pingdotgg/t3code by fast-forwarding main and rebasing the custom branch, resolving conflicts against the FORK.md customization ledger. Use when asked to sync, update, or rebase onto upstream, or when custom has fallen behind upstream/main.
---

# 同步上游

目标：`main` 快进到 `upstream/main`，再把 `custom` rebase 到新的 `main` 上，同时保证定制清单里的每个功能都还能用。

## 1. 准备

```sh
git status --short          # 必须干净；有改动先问开发者怎么处理
git fetch upstream --prune
git log --oneline custom..upstream/main | wc -l   # 这次要吃进多少上游提交
```

先停掉这个 checkout 启动的开发服务器（用你自己 harness 的停止功能，不要按进程名去杀）。

## 2. 更新 main

```sh
git switch main
git merge --ff-only upstream/main
```

如果无法快进，说明有人在 `main` 上提交过。停下来告诉开发者，不要强行处理。

## 3. Rebase custom

```sh
git tag backup/custom-$(date +%Y%m%d-%H%M%S) custom   # 本地备份，出问题可以随时回退
git switch custom
git rebase main
```

先读 [FORK.md](../../../FORK.md) 的「定制清单」，弄清楚每个定制提交想达到什么效果。解冲突的原则：

- 上游的结构为准，再把我们的意图重新落上去。不要为了保住我们的写法去回滚上游的改动。
- `AGENTS.md` 冲突：保留上游全文，再把文件末尾的「Fork 层」小节原样接回去。
- 上游新增的迁移编号和我们用过的编号重叠，或者某个定制已被上游原生实现：停下来告诉开发者，并给出建议（drop 这个定制，还是改写）。
- `pnpm-lock.yaml` 冲突：取上游版本，rebase 完成后重新跑 `vp i`。

## 4. 验证

```sh
vp i
vp run check:changed main         # 只检查定制提交相对 main 的部分
```

再按定制清单里列出的文件跑对应测试（`vp test run <files>`），最后 `vp run dev:desktop` 实际把每个定制点一遍。不要跑全仓检查（`vp check`、`vp run -r test`）。

## 5. 收尾

- 在 FORK.md 的「同步记录」里加一行：日期 + `git rev-parse --short main`。如果某个定制被改写或 drop 了，同步更新定制清单。
- 推送需要开发者明确同意。`main` 正常 push 即可；rebase 后的 `custom` 需要 `git push --force-with-lease origin custom`，属于强推，**必须先征得同意**。
- 向开发者汇报：吃进了多少上游提交、解了哪些冲突、哪些定制有变化、验证结果如何。
