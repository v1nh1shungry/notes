原文：https://andrewlock.net/super-charging-git-rebase-with-git-absorb/

# Fixup Commit

场景是开发分支有多个提交，但需要将修改 amend 到“上游提交”。例如，示例仓库有3个提交
```bash
$ git log --oneline
0a61723 (HEAD -> main) third commit
fb585f1 second commit
49af51b first commit
```
需要修改 second commit。

可以直接在当前工作目录中做修改，当修改完毕后，将对应的修改 `git add` 到暂存区，再使用 `git commit --fixup <要 amend 的提交的 SHA>` 
```bash
$ git commit --fixup fb585f14125a8d78b1c9f690ec4cbf4f50ad3812
[main e1f7584] fixup! second commit
 1 file changed, 1 insertion(+), 1 deletion(-)
```
将生成一个 fixup commit，可以观察到提交信息的格式为 `fixup! <要 amend 的提交的提交信息`。

将对应修改依次提交到对应的 fixup commit 后，使用 `git rebase --autosquash`，将正常启动 rebase 流程，但是会自动将 fixup commit 重排并 squash 至对应的提交
```gitrebase
pick fb585f1 second commit
fixup e1f7584 fixup! second commit
pick 0a61723 third commit
```

# `git absorb`

`git absorb` 命令是一个 [Git 插件](https://github.com/tummychow/git-absorb)。它的作用是自动将对应修改生成对应的 fixup commit 而无需手动指定 fixup 的目标。

只需要将所有修改添加到暂存区，再使用 `git absorb`，所有已暂存的修改将被分配至对应的 fixup commit，可以在这个阶段进行检查。

也可以直接 `git absorb --and-rebase`，相当于在 `git absorb` 后自动运行 `git rebase --autosquash` 合并所有 fixup commit。