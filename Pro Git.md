* `git status -s` 简洁展示仓库当前状态；
* `git rm --cached` 将文件从 Git 仓库中删除，但在磁盘中保留；
* 删除远程分支/标签的两种方式：
	*  `git push origin :<TARGET>`
	* `git push --delete <TARGET>`
* 在 alias 中，在命令前添加 `!` 来运行外部命令；
* `git clone --bare` 来克隆一个**裸仓库**，即只包含 `.git` ，不包含当前工作目录的仓库；
* `git rev-parse` 可用于查看某个分支或者某段 SHA-1 简写所指向的完整 SHA 值；

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

| 选项  | 说明                                          |
|-------|-----------------------------------------------|
| `%H`  | 提交的完整哈希值                              |
| `%h`  | 提交的简写哈希值                              |
| `%T`  | 树的完整哈希值                                |
| `%t`  | 树的简写哈希值                                |
| `%P`  | 父提交的完整哈希值                            |
| `%p`  | 父提交的简写哈希值                            |
| `%an` | 作者名字                                      |
| `%ae` | 作者的电子邮件地址                            |
| `%ad` | 作者修订日期（可以用 --date=选项 来定制格式） |
| `%ar` | 作者修订日期，按多久以前的方式显示            |
| `%cn` | 提交者的名字                                  |
| `%ce` | 提交者的电子邮件地址                          |
| `%cd` | 提交日期                                      |
| `%cr` | 提交日期（距今多长时间）                      |
| `%s`  | 提交说明                                      |

* `git log --since=2.weeks` 列出最近两周的所有提交；
* **`git log -S <STR>` 传说中的 pickaxe ！显示包含指定字符串的所有提交；**
* `git log --no-merges` 可过滤掉合并提交；
* `git log --abbrev-commit` 为 SHA-1 值生成简短且唯一的缩写，默认使用7个字符；
* `git log --show-signature` 查看并验证签名；
* **`git log -L :foobar:foobar.c` 可查看 `foobar.c` 文件中的 `foobar` 函数的变更历史**；

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
* `git shortlog` 可生成简易的 changelog 文档，包括所有提交的概要信息，按作者分组；`-sne` 仅显示作者名字、邮箱和提交数量；

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

# `reflog`

* `reflog` 用于记录**本地仓库**的 `HEAD` 和分支引用的指向历史；
* 使用 `@{n}` 格式来引用 `git reflog` 中输出的提交记录；
* `git log -g` 可输出对应的 `reflog` 信息；

# 祖先引用

* 在引用后添加 `^` 或 `~` 表示该引用的祖先引用；
* 对于合并提交，可以在 `^` 添加数字来指明**哪一个父提交**，因为只有合并提交具有多个父提交；
* `HEAD~2` 代表**父提交的父提交**；
* `^` 和 `~` 的区别在于后面加数字的含义，`^` 是沿着合并提交的父提交横向遍历，而 `~` 则沿着提交历史纵向遍历；

# 提交区间

* 双点
	* 在两个引用之间添加 `..` 表示在后者而不在前者的提交，当后者留空时默认为 `HEAD`；
	* 常用场景
		* `git log experiment..main` 在 `rebase` 到 `main` 分支最新进度前查看即将合并的内容；
		* `git log origin/main..HEAD` 在推送分支前查看即将推送的内容；
* 多点
	* 双点语法只能指定两个引用；
	* 以下三条命令等价
```bash
$ git log refA..refB
$ git log ^refA refB
$ git log refB --not refA
```
* 三点
	* 在两个引用之间添加 `...` 表示被两个引用**之一**包含但又不被两者同时包含的提交；
	* `--left-right` 参数将显示每个提交属于哪一侧的分支；

# 交互式暂存

* `git add -i` 可进入交互式暂存模式，在希望将大量修改分割成若干提交、暂存文件的部分修改时很方便；
	* `update` 暂存，`revert` 取消暂存；
	* `diff` 查看**已暂存**的修改（类似 `git diff --cached`）；
	* 使用 `patch` 命令来暂存文件的部分修改；
* 也可使用 `git add -p` 来进行文件部分暂存，与之相对的可以使用 `git reset -p` 来部分重置，`git checkout -p` 来部分 `checkout` 以及 `git stash -p` 来部分 `stash` 文件；

# `git stash`

* `git stash apply` 将应用栈顶的修改，而 `git stash pop` 则会在应用后将修改从栈中丢弃；
* `git stash apply` **不会重新暂存已暂存的修改**，使用 `--index` 来尝试重新应用暂存；
* `git stash --keep-index` 将只 `stash` 未暂存的修改，保留已暂存的内容；
* `git stash -u` 来 `stash` 未跟踪的文件，`git stash -a` 来 `stash` 已忽略的文件；
* `git stash branch` 用于从进行 `stash` 时所在的提交中创建新的分支，在新分支中应用 `stash` 的修改，并从栈中丢弃修改，在重新应用 `stash` 的修改可能导致冲突时很方便；

# 清理工作目录

* `git clean` 将删除所有**未被跟踪和未被忽略**的文件，`-d` 参数将删除空的子目录；
* **推荐在运行该命令前执行 `git stash -a`，避免误删除的文件无法恢复**；
* `git clean -n` 执行一次 dry-run ，仅输出该命令将删除哪些文件而不实际执行；
* `git clean -x` 将额外删除已忽略的文件；

# 签署认证

* `git tag -s <NAME> -m <MESSAGE>` 使用 `-s` 替代 `-a` 即可签署一条新的标签；
* `git tag -v <NAME>` 验证签名，签署者的公钥需要在 GPG 的 key-list 中；
* `git commit` 和 `git merge` 可使用 `-S` 签署提交；
* `git merge` 和 `git pull` 可使用 `--verify-signatures` 来验证签名并拒绝没有携带可信签名的提交；

# `git grep`

* `-n` 输出匹配行的行号；
* `-p` 显示每一个匹配项所在的上下文函数；
* 使用 `--and` 来组合多个查询字符串，确保多个匹配出现在同一文本行；
* 输出将按照所在文件分组，`--heading` 将在分组前输出文件名，`--break` 将输出额外的空行分割各个分组；

# 修改历史提交

* `git commit --amend --no-edit` 可跳过编辑提交日志；
* `git commit --amend` 只能修改最近一次提交，若需要修改更远的提交，需使用 `git rebase -i`
	* 假设需要修改最近的第三条提交，则需要指定需要修改的提交的父提交，即 `git rebase -i HEAD~3`;
	* Git 将提供一个即将运行的脚本，**注意：脚本显示的提交序列从上到下依次从旧到新，因为 Git 即将回溯到 `HEAD~3` 并依次按照提交前的命令重新将各提交应用**。通过修改该脚本来指定需要的操作，将第三条提交的命令修改为 `edit`；
	* 退出编辑后，Git 将回溯到 `HEAD~3`，并开始执行脚本的命令。将应用第三次提交后暂停，此时就可以通过 `git commit --amend` 来修改该提交；
* **`git rebase -i <COMMIT>` 的本质就是用户编辑 Git 提供的脚本，在用户编辑完后，回到 `<COMMIT>` ，并依次执行脚本上的命令，重新将脚本上的提交应用，因此删除掉脚本上的一行提交就会删除掉这个提交，调换脚本上提交的顺序就会调换两个提交的应用顺序**；
* 拆分提交
```bash
# 拆分最近的第三次提交
$ git rebase -i HEAD~3
# 将最近的第三次提交的命令修改为 edit
$ git reset HEAD^
# 暂存需要提交的修改
$ git commit -m "first split commit"
# 暂存需要提交的修改
$ git commit -m "second split commit"
$ git rebase --continue
```
* **谨慎使用 `git filter-branch`**，考虑使用 [git-filter-repo](https://github.com/newren/git-filter-repo)
	* `git filter-branch --tree-filter 'rm -f dummy.txt' HEAD` 从所有提交中删除误提交的文件，`--tree-filter` 将在 `checkout` 每一个提交后运行指定的命令然后重新提交，可使用 `--all` 在所有分支上运行；
	* `git filter-branch --subdirectory <SUBDIR> HEAD` 将子目录作为新的根目录
	* 修改邮箱地址
```bash
git filter-branch --commit-filter ' \
  if [ "$GIT_AUTHOR_EMAIL" = "hongyuanjing@scutech.com" ]; \
  then \
    GIT_AUTHOR_NAME="Yuanjing Hong"; \
    GIT_AUTHOR_EMAIL="v1nh1shungry@outlook.com"; \
    git commit-tree "$@"; \
  else \
    git commit-tree "$@"; \
  fi' HEAD
```

# `reset` 和 `checkout`

* Git 维护三个状态：`HEAD` 、索引和工作区，分别对应分支、暂存区和当前工作目录的文件；通过 `git add` ，将工作区对应文件复制到索引，通过 `git commit` ，将当前索引的状态生成为一个快照，将 `HEAD` 指向该快照；`git diff` 将输出工作区状态和索引状态的差异，`git diff --cached` 将输出索引状态和 `HEAD` 指向的快照的差异；
* `git reset <COMMIT>` 将改变 `HEAD` 的指向
	* `git reset --soft HEAD^` 保持索引与工作区的状态不变，将 `HEAD` 指向上一次提交，因此效果就是将当前提交撤销，而当前提交做出的修改依然在暂存区中；
	* `git reset --mixed` 在 `git reset --soft` 基础上，还将用 `HEAD` 指向的快照的内容更新索引，因此效果就是将当前提交撤销，并将提交做出的修改取消暂存；
	* `git reset --hard` 在 `git reset --mixed` 基础上，**强制覆盖工作目录中的文件，是会销毁修改的危险操作**；
	* 压缩提交
```bash
# 假设有 A - B - C 三个提交，希望保留 A 和 C 的提交历史
# 当前 HEAD 指向 C
# 切换到希望压缩的提交的父提交
$ git reset --soft HEAD~2
# 当前 HEAD 指向 A，而索引和工作区的状态仍然是 C 的状态
# 因此再直接提交即可将 B 的历史压缩掉
$ git commit
```
* `git reset <PATH>` 不会改变 `HEAD` 的指向，而是会将 `HEAD` 指向的快照内容更新索引，因此效果就是取消暂存文件，在新版本中添加了 `git restore --staged` 命令旨在替代 `git reset` 的这部分功能；
* `git checkout <COMMIT>` 的效果与 `git reset --hard <COMMIT>` 类似，但是前者是**安全的**，不会丢失已做出的更改。此外，**`git checkout` 只会修改 `HEAD` 的指向，而 `git reset` 将同步修改分支的指向**，在新版本中添加了 `git switch` 用于替代 `git checkout` 的这部分功能；
* `git checkout <PATH>` 的效果类似 `git reset --hard <PATH>`，除了用 `HEAD` 指向的快照内容更新索引之外，还将更新工作目录的文件，因此效果就是撤销文件修改，在新版本中添加了 `git restore` 来替代 `git checkout` 的这部分功能；

# 合并冲突

* 合并策略可以带有参数
	* `-Xignore-all-space` 将在合并时**完全忽略**空白修改，而 `-Xignore-space-change` 将一个空白字符和多个连续空白字符视作等价；
	* `-Xours` 或 `-Xtheirs` 将在合并出现冲突时直接选择一边的修改并丢弃另一边；
* 在一次三路合并中，`:1:<FILE>` 是共同的祖先版本（即两条分支的分叉点），`:2:<FILE>` 是当前分支的版本（ours），`:3:<FILE>` 是即将合并的版本（theirs），可以使用 `git show` 来查看这些版本的内容；
	* `git ls-files -u` 来列出未合并的文件和对应的 SHA-1 值，`:1:<FILE` 只是查找对应 SHA-1 值的语法；
* `git diff --ours` 可在 `git add` 前查看冲突解决后的结果与当前分支的差异，`--theirs` 则比较与即将合并如的分支的差异，而 `--base` 则比较与共同的祖先版本的差异；
* 在合并冲突时，默认的合并冲突标记只显示 ours 和 theirs 的上下文，可使用 `git checkout --conflict=diff3 <FILE>` 替换合并标记，添加共同祖先版本的上下文；
* 在合并冲突中，可以使用 `git checkout --ours` 或 `--theirs` 快速选择其中一个版本的修改，丢弃另一个版本；
* `git log --oneline --left-right --merge -p` 可查看合并冲突中任何一边接触了冲突文件的提交；
* `git log --cc -p` 可查看合并提交是如何解决冲突的；
* `git revert -m 1 HEAD` 来还原一个合并提交，一个合并提交具有两个父节点，`-m 1` 参数表示属于当前分支的父节点的内容，撤销由合并的分支引入的修改；使用这种方法撤销合并，当需要再次合并相同分支时，需要**先将撤销合并提交时生成的提交撤销 `git revert <REVERT_COMMIT>`，再进行合并**；
* `git merge -s ours` 将进行一次**假合并**，生成一次以两边分支为父提交的合并提交，但不合并入任何修改，即合并前后代码没有变化；
> 当再次合并时从本质上欺骗 Git 认为那个分支已经合并过经常是很有用的。 例如，假设你有一个分叉的 `release` 分支并且在上面做了一些你想要在未来某个时候合并回 `master` 的工作。 与此同时 `master` 分支上的某些 bugfix 需要向后移植回 `release` 分支。 你可以合并 bugfix 分支进入 `release` 分支同时也 `merge -s ours` 合并进入你的 `master` 分支 （即使那个修复已经在那儿了）这样当你之后再次合并 `release` 分支时，就不会有来自 bugfix 的冲突。

# `git blame`

* `git blame` 的输出中，开头带有 `^` 标记的行代表该行自该文件第一次提交后从未修改；
* `git blame -C` 将尝试找出文件中从别的地方复制过来的代码片段的出处；

# 二分查找

* 手动流程
	1. `git bisect start` 开始进行二分查找；
	2. `git bisect bad` 标记目前所在的提交有问题；
	3. `git bisect good <COMMIT>` 标记已知的最后一次正常提交；
	4. Git 将 `checkout` 至中间的提交，测试当前提交是否正常，并根据结果执行 `git bisect good/bad`；
	5. 重复执行上一步直至 Git 找出出问题的提交；
	6. `git bisect reset` 将重置 `HEAD` 到最开始的位置；
* 当有工具脚本能帮助判断当前提交是否正常时，可使用自动流程
	1. `git bisect start <BAD_COMMIT> <GOOD_COMMIT>` 设定不正常提交到正常提交的范围；
	2. `git bisect run test.sh` Git 将自动在每个 `checkout` 的提交中运行 `test.sh`；

# 子模块

* `git submodule add <URL> [<PATH>]` 添加子模块，默认将子模块放在与子项目同名的目录中，可使用可选的路径参数指定路径；
* 克隆含有子模块的仓库
	* 克隆仓库后，执行 `git submodule init` 初始化本地配置文件，再执行 `git submodule update` 抓取子模块的数据并 `checkout` 到父项目指定的提交；
	* `git clone --recurse-submodules` 自动初始化并更新子模块，包括嵌套子模块；如果在克隆时忘记添加 `--recurse-submodules` 参数，可以执行 `git submodule update --init`，如果有嵌套子模块，则还要加上 `--recursive` 参数；
* `git diff --submodule` 可 pretty-print 子模块的更新情况，包括更新的提交列表，可将 `diff.submodule` 设置为 `log` 来作为默认行为而不需要添加 `--submodule` 参数；
* `git submodule update --remote` 自动进入子模块并更新，更新跟踪的分支由 `.gitmodules`（公共的）或 `.git/config`（本地的）中的 `submodule.<SUBMODULE>.branch` 决定；
* 设置 `status.submodulesummary` 为 1 可在 `git status` 显示子模块的更改摘要；
* 更新含有子模块的仓库
	* `git pull` 会递归抓取子模块的更改，但**不会更新**子模块，需要再运行 `git submodule update --init --recursive`；
	* `git pull --recursive-submodules` 将在拉取后自动更新子模块；可以设置 `submodule.recurse` 为 `true` 来将其作为默认行为；
* 当子模块的 URL 改变时
```bash
# 更新本地配置中的子模块 URL
$ git submodule sync --recursive
$ git submodule update --init --recursive
```
* 在子模块中工作
	* 更新子模块时，Git 将自动 `checkout` 到指定的提交，处于 `HEAD` 分离的状态，在这种情况下在子模块中进行的工作很容易丢失，需要先 `checkout` 到工作分支中；
	* 在更新子模块时，使用 `git submodule update --remote --merge` ，Git 将自动抓取更新并将更新合并入当前所在的分支中；
	* 当本地子模块提交了一些修改，而子模块上游也有修改时，需要运行 `git submodule update --remote --rebase` 来将上游修改并入本地；
	* 如果忘记添加 `--merge` 或者 `--rebase`，Git 会再次将子模块 `checkout` 至指定提交，只需再 `checkout` 至工作分支即可，工作分支上已提交的修改不会丢失，然后再手动合并即可；
	* 如果子模块中还有未提交的修改，子模块更新将只抓取更新而不尝试合并，直接终止更新；
	* 当子模块有本地修改但尚未提交时，该改动仅在本地可见，其他人在拉取项目时将因无法拉取子模块更新而无法更新；可使用 `git push --recurse-submodules=check` 在推送父项目时检查所有子模块是否含有未推送的更改；`git push --recurse-submodules=on-demand` 则在推送时尝试推送所有子模块中未推送的更改；可将 `push.recurseSubmodules` 设置为对应值；
	* 当子模块的历史已经分叉并且在父项目中分别提交到了分叉的分支上且存在冲突时，`git pull` 将发生合并冲突
```bash
$ cd DbConnector
# 获取试图合并的两个分支的提交 SHA-1 值
$ git diff
diff --cc DbConnector
index eb41d76,c771610..0000000
--- a/DbConnector
+++ b/DbConnector
# eb41d76 是本地分支所在的提交，c771610 是尝试合并入的分支所在的提交
# 手动创建对应分支
$ git branch try-merge c771610
# 合并
$ git merge try-merge
# 解决冲突
$ git commit
# 回到父项目
$ cd ..
# 提交合并
$ git add DbConnector
$ git commit
```
* `git submodule foreach '<COMMAND>'` 遍历每一个子模块并运行指定命令；
* `git checkout` 在切换分支时，默认不会更新子模块，可添加 `--recursive-submodules` 参数在切换分支时自动将子模块更新到正确的状态；
* 将子目录转换为子模块
	* 需要先取消跟踪子目录，再添加子模块；
	* 假设在当前分支下进行了转换工作，在切换回尚为子目录的分支时会出错，可使用 `git checkout -f` 强制切换，**注意会覆盖未提交的修改**；再切换回已转换的分支时，子模块目录为空，需要到子模块目录运行 `git checkout .` 来恢复文件；

# 打包

* `git bundle create repo.bundle HEAD main` 将 `main` 分支打包为 `repo.bundle` 文件，**注意添加 `HEAD` 引用标注 `checkout` 的位置**；
* `git clone repo.bundle repo` 从 Git 包中克隆出仓库，如果在打包时没有指定 `HEAD`，需要使用 `-b main` 指定要 `checkout` 的分支；
* `git bundle create commits.bundle main ^origin/main` 将本地 `main` 分支中添加的提交打包；
* `git bundle verify` 检查是否为合法的 Git 包，是否拥有共同的祖先；
* `git bundle list-heads` 可查看 Git 包中的可导入的分支；
* `git fetch ../commits.repo feature1:feature1` 从 Git 包中导入 `feature1` 分支；