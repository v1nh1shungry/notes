* `git status -s` 简洁展示仓库当前状态；
* `git rm --cached` 将文件从 Git 仓库中删除，但在磁盘中保留；
* `git reset HEAD <FILE>` 等价于 `git restore --staged <FILE>`；
* 删除远程分支/标签的两种方式：
	*  `git push origin :<TARGET>`
	* `git push --delete <TARGET>`
* 在 alias 中，在命令前添加 `!` 来运行外部命令；
* `git clone --bare` 来克隆一个**裸仓库**，即只包含 `.git` ，不包含当前工作目录的仓库；

# `git diff`

* `git diff --check` 可检查存在的行尾空格；
* 当希望查看当前分支与 `main` 分支合并时的 `diff` 时，`git diff main` **并不总能**得到正确的结果。当当前分支直接延续自 `main` 分支时，该命令能输出正确结果，但当 `main` 分支上有当前分支尚未包含的提交时，该命令将输出当前分支将这些提交删除的结果。**正确的做法是先找出两者的公共祖先，再对该提交进行 `diff`**
```bash
$ git diff $(git merge-base <BRANCH> main)
```
Git 还提供了等价的便捷方式
```bash
$ git diff main...<BRANCH>
```

# `git checkout`

* `git checkout <FILE>` 可还原已修改的文件，在新版本的 Git 中，添加了 `git restore` 命令用于接管这部分功能；
* `git checkout <BRANCH>` 可切换分支，在新版本的 Git 中，添加了 `git switch` 命令用于接管这部分功能；

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
* `git log <BRANCH-A>..<BRANCH-B>` 只显示所有在 `<BRANCH-B>` 但不在 `<BRANCH-A>` 的提交；
* `git log --not main` 查看当前分支上所有 `main` 分支尚未包含的提交；

# 标签

* `git tag -a <NAME> -m <MESSAGE>` 添加附注标签，附注标签是一个完整的 Git 对象，具有作者名字、邮箱等信息，以及标签信息（相当于 commit 的提交日志），就像一个特殊的提交；
* `git tag <NAME>` 添加一条轻量标签，轻量标签是指向某个提交的引用，相当于一个不会改变的分支；
* 通常建议添加附注标签；
* `git push --tags <REMOTE_NAME>` 推送所有不在远程仓库的标签；

# 发布

* **`git describe` 将为提交生成一个可读的名称，由最近的标签名、自该标签之后的提交数和该提交的部分 SHA-1 值组成；**
* 使用 `git archive` 将为项目打包压缩包，使用 `--prefix` 参数指定压缩包解压后的文件夹前缀，**注意文件夹名后的 `/` 不要漏**
```bash
# 默认的打包格式为 tar
$ git archive main --prefix='project/' | gzip > $(git describe).tar.gz

# 可指定创建一个 zip 压缩包
$ git archive main --prefix='project/' --format=zip > $(git describe).zip
```
* `git shortlog` 可生成简易的 changelog 文档，包括所有提交的概要信息，按作者分组；

# 分支

* `git branch -v` 也能查看各个分支当前所指的提交， `git branch -vv` 则还能查看各个分支的远程跟踪分支情况；
* `git branch --merged` 可查看已经合并到当前分支的分支， `--no-merged` 则查看尚未合并的分支；
* 尚未合并到当前分支的分支，无法使用 `git branch -d` 删除，需使用 `git branch -D` 强制删除；
* `git checkout -b <BRANCH> <REMOTE>/<BRANCH>` 来建立远程分支的用于工作的本地分支，此外，还可用 `git checkout --track <REMOTE>/<BRANCH>` 来简化这一操作，最后，当本地不存在尝试签出的分支时， `git checkout <BRANCH>` 等价于上述操作；
* `git branch -u <REMOTE>/<BRANCH>` 来显示设置跟踪的远程分支；
* **可使用 `@{upstream}` 或 `@{u}` 来引用分支的上游分支**；
* 保持 fork 仓库与上游同步
```bash
# 添加上游远程仓库
$ git remote add upstream https://github.com/v1nh1shungry/notes.git
# 设置远程跟踪分支，使 main 分支从 upstream 抓取更新
$ git branch -u upstream/main main
# 设置默认推送仓库为 origin，使 main 分支将更新推送至 origin
$ git config --local remote.pushDefault origin
```

# Github

* Github 将 PR 请求分支视为一种“假分支”，因此即便 PR 的源分支来自其他用户的 fork 仓库，在上游仓库仍然能引用到这些分支
	* `git ls-remote` 列出远程仓库上的所有引用。如果仓库在 Github 上有打开的 PR ，将会得到一些以 `refs/pull/` 开头的引用，它们实际上是分支，但因为不在 `refs/heads/` 中，因此在 `fetch` 过程中将被忽略；
	* 直接抓取和合并 PR 请求分支，而无需将 fork 仓库添加为远程仓库
```bash
# 抓取 #1 分支，以 FETCH_HEAD 指向该分支
$ git fetch origin refs/pull/1/head
# 将 #1 分支合并
$ git merge FETCH_HEAD
```
* 抓取**所有的** PR 分支并保持更新
```gitconfig
# 编辑 .git/config
[remote "origin"]
	url = https://github.com/v1n1h1shungry/notes.git
	# 将 refs/heads/ 下的内容放在本地仓库的 refs/remotes/origin 下
	fetch = +refs/heads/*:refs/remotes/origin/*
	# 添加一行，将 refs/pull/ 下的 head 放在本地仓库的 refs/remotes/origin/pr 下
	fetch = +refs/pull/*/head:refs/remotes/origin/pr/*
```
现在运行 `git fetch` ，PR 请求将被抓取并以 `pr/*` 分支引用；
* `refs/pull/*/head` 代表该 PR 的分支 HEAD，而 `refs/pull/*/merge` 则代表 Github 自动合并对应的提交记录，可通过该引用在实际合并之前就进行测试；

# `rebase`

* `git rebase --onto main server client` 在 `client` 分支上，**找出它与 `server` 分支分歧处之后的提交（即从 `server` `checkout` 后的提交）**，将其应用在 `master` ；
* 通过合并产生并已推送的提交，就不要再使用 `rebase` 整理，当这种情况发生时， `git pull --rebase` 很可能能自动解决。**本质上一条原则——多人合作的分支上，不要进行 `git push -f`，当你需要执行这条命令才能推送提交时，就已经错了。**

# 协议

* 本地协议中，直接使用路径，Git 会尝试硬链或者复制文件；若在 URL 开头指定 `file://`，则会使用平时用于网络传输的进程，传输效率会低，指定 `file://` 的目的主要是取得一个没有外部引用或对象的干净副本；
* HTTP 协议支持匿名读取 Git 仓库，而 SSH 协议则不支持，这是 SSH 协议主要的缺陷；
* SSH 协议
	* 用户通过 SSH 连接到 Git 服务器，并对某项目目录有写权限的话，他将自动获得推送权限。如果在一个项目目录中执行 `git init --shared` ，Git 会自动修改该仓库目录的组权限为可写，**注意：该命令不会修改、破坏任何提交等内容；**
	* 如果希望多个用户能读写 Git 服务器，但又不希望为每个人单独创建用户，则可以使用 SSH 协议，创建一个 `git` 用户，将需要访问的用户的公钥写入 `git` 用户的 `authorized_keys` 中，从而使已认证的用户能以 `git` 用户读写项目；
	* `git-shell` 是一个能将用户活动限制在 Git 相关的范围内的 shell 工具，可通过 `chsh git -s $(which git-shell)` 来指定 `git` 用户登录时使用的 shell 为 `git-shell`；
* Git 协议
	* 没有任何鉴权机制，唯一的优势就是快速；
	* 通过在仓库下创建 `git-daemon-export-ok` 文件来开放该仓库的 Git 协议访问；

# 补丁

* `git apply` 与 `patch -p1` 命令的区别
	* 前者能处理文件添加、删除和重命名操作，而后者不能；
	* 前者采取类似事务的模型，即补丁只有全部内容应用和完全不应用两种结果，而后者则可能导致补丁部分应用；
* `git apply --check` 可以检查补丁是否能被顺利应用，而不实际应用补丁；
* `git diff` 产生的补丁只能用 `git apply` 来应用，而 `git format-patch` 产生的补丁则要用 `git am` 来应用；

# `git rerere`

* `rerere` (reuse recorded resolution) 将记录解决合并冲突的修复方案，当下次遇到相似的合并冲突时，Git 将自动应用记录的修复方案而无需用户干预；
* `git config --global rerere.enabled true` 启用 `rerere` 功能；
* 在没有全局启用 `rerere` 时，执行 `git rerere` 将寻找当前任一冲突相关的修复方案并解决冲突；