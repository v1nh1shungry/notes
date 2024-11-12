原文链接：https://github.blog/open-source/git/get-up-to-speed-with-partial-clone-and-shallow-clone/

# Quick Summary

* `git clone --filter=blob:none <URL>` 创建一个 blobless clone，即下载所有可达的 commit object 和 tree object，而只按需下载 blob object，**开发人员推荐**；
* `git clone --filter=tree:0 <URL>` 创建一个 treeless clone，即下载所有可达的 commit object，而只按需下载 tree object 和 blob object，**需要访问提交历史的一次性构建环境推荐**；
* `git clone --depth=1 <URL>` 创建一个 shallow clone，即通过截断提交历史来减小 clone 体积，**不推荐开发人员使用，无需提交历史的一次性构建环境推荐**；

# blobless clone

* 在 `git clone` 时仅下载所有 commit object 和 tree object，而仅在 `git checkout` 时下载所需的 blob object，这包括 `git clone` 中的 `checkout` 操作；
* `git fetch` 也只会下载新的 commit object 和 tree object，新的 blob object 同样只有在 `git checkout` 时才会下载；
* `git log` 这种不需要内容的命令不会触发 blob object 的下载，`git diff` 这种需要内容的命令会在第一次运行时触发下载；
* blobless clone 拥有完整的提交历史，仅有部分命令会变得更慢，因此是最推荐的 partial clone 形式；

# treeless clone

* `git checkout` 到一个 tree object 缺失的提交时，会下载该提交的 tree object，以及该 tree object 中所有可达的 tree object，此外，客户端没有途径通知服务器本地已有的 tree object，因此可能会发送许多本地已有的 tree object；
* treeless clone 可能显著提高 clone 的速度，但是将影响更多的命令的使用，搭建一次性的构建环境将是一个不错的选择；
* 在拥有 submodule 的仓库使用 treeless clone 的表现更差，`git config fetch.recurseSubmodules false` 可能改善；

# shallow clone

* 最好搭配 `--single-branch --branch=<BRANCH>` 使用，确保只下载要使用的提交；
* `git log`、`git blame` 等需要提交历史的命令将无法正常使用；