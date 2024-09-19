* `git status -s` 简洁展示仓库当前状态；
* `git rm --cached` 将文件从 Git 仓库中删除，但在磁盘中保留；
* `git reset HEAD <FILE>` 等价于 `git restore --staged <FILE>`，`git checkout <FILE>` 等价于 `git restore <FILE>`；
* 删除远程分支/标签的两种方式：
	*  `git push origin :<TARGET>`
	* `git push --delete <TARGET>`
* 在 alias 中，在命令前添加 `!` 来运行外部命令；

# `.gitignore`

* 匹配模式以 `/` 开头防止递归；
* 匹配模式以 `!` 开头表示排除该模式（即不要忽略该模式）；
* `glob` 模式要点
	* `*` 匹配任意个任意字符，`?` 匹配一个任意字符；
	* `[abc]` 匹配任意一个 `[]` 内的字符，`[a-c]` 匹配任意一个 `[]` 内所示范围的字符；
	* `**` 匹配任意目录；
* `man gitignore`
* 子目录可拥有自己的 `.gitignore` ，仅在子目录中生效；

# `git log`

* `git log --pretty=format:"<FORMAT>"` 可指定 `git log` 显示的提交信息格式；
* `git log --since=2.weeks` 列出最近两周的所有提交；
* **`git log -S <STR>` 传说中的 pickaxe ！显示包含指定字符串的所有提交；**
* `git log --no-merges` 可过滤掉合并提交；
* `git log --decorate` 查看各个分支当前所指的提交；

# 标签

* `git tag -a <NAME> -m <MESSAGE>` 添加附注标签，附注标签是一个完整的 Git 对象，具有作者名字、邮箱等信息，以及标签信息（相当于 commit 的提交日志），就像一个特殊的提交；
* `git tag <NAME>` 添加一条轻量标签，轻量标签是指向某个提交的引用，相当于一个不会改变的分支；
* 通常建议添加附注标签；
* `git push --tags <REMOTE_NAME>` 推送所有不在远程仓库的标签；

# 分支

 * `git branch -v` 也能查看各个分支当前所指的提交， `git branch -vv` 则还能查看各个分支的远程跟踪分支情况；
 * `git branch --merged` 可查看已经合并到当前分支的分支， `--no-merged` 则查看尚未合并的分支；
 * 尚未合并到当前分支的分支，无法使用 `git branch -d` 删除，需使用 `git branch -D` 强制删除；
 * `git checkout -b <BRANCH> <REMOTE>/<BRANCH>` 来建立远程分支的用于工作的本地分支，此外，还可用 `git checkout --track <REMOTE>/<BRANCH>` 来简化这一操作，最后，当本地不存在尝试签出的分支时， `git checkout <BRANCH>` 等价于上述操作；
 * `git branch -u <REMOTE>/<BRANCH>` 来显示设置跟踪的远程分支；
 * **可使用 `@{upstream}` 或 `@{u}` 来引用分支的上游分支**；

# `rebase`

* `git rebase --onto main server client` 在 `client` 分支上，**找出它与 `server` 分支分歧处之后的提交（即从 `server` `checkout` 后的提交）**，将其应用在 `master` ；