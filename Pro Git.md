* `git status -s` 简洁展示仓库当前状态；
* `git rm --cached` 将文件从 Git 仓库中删除，但在磁盘中保留；
* `git reset HEAD <FILE>` 等价于 `git restore --staged <FILE>`，`git checkout <FILE>` 等价于 `git restore <FILE>`；
* `git push --tags <REMOTE_NAME>` 推送所有不在远程仓库的标签；
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