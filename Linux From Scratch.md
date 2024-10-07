# 前置知识

## [Software-Building-HOWTO](https://tldp.org/HOWTO/Software-Building-HOWTO.html)

* `a.out` 是旧版类 UNIX 系统中的可执行二进制文件的格式，后被 ELF 格式替代；

### 压缩与解压缩

* 通常有两级，第一级 `.tar`，第二级通常有两种格式，`.gz` 和 `.bz2`，分别对应 `gzip` 和 `bzip2`；
* 解压 `.tar.gz`，**注意 `-` 表示从 `stdin` 中读取**
	* `tar xzvf <FILENAME>`
	* `gzip -cd <FILENAME> | tar xvf -`
	* `gunzip -c <FILENAME> | tar xvf -`
* 解压 `.tar.bz2`
	* `tar xyvf <FILENAME>`
	* `bzip2 -cd <FILENAME> | tar xvf -`
* tar 还可以使用 `-I <COMMAND>`、`--bzip2`、`--gzip` 等参数指定使用的解压缩格式；

### 安装

* 在安装一个二进制包前，应当对其进行校验
```bash
# RPM
$ rpm --checksig <PACKAGE>
$ rpm -K --nopgp <PACKAGE>

# DEB
$ dpkg -I <PACKAGE>
$ dpkg -e <PACKAGE>
```

## [Beginner's Guide to Installing from Source](https://moi.vonos.net/linux/beginners-installing-from-source/)

### 校验安装包

* MD5
	* `md5sum <FILE>` 计算单个文件的 MD5；
	* `md5sum -c <MD5SUMS>` 根据软件供应者提供的 md5sums 文件，文件中包含文件名-校验和列表，查找列表中的每个文件并校验；
* GPG 签名
	1. 下载软件供应者的公钥（**尽量使用 HTTPS 协议下载**）
	2. 下载软件包文件对应的签名文件，通常与软件包同名并以 `.sig` 或 `.asc` 为后缀；
	3. `gpg --import <PUBLIC-KEY>` 导入供应者的公钥；
	4. `gpg --verify <SIGNATURE>` 验证签名；

### 归档与压缩

* tar **不是**压缩软件，而是**归档软件**，用于将多个文件**打包**成一个文件；tar 的一个优点是保留了文件权限；**tar 不会压缩文件内容**，因此通常在将多个文件打包成 `.tar` 文件后会再额外使用 gzip 或者 bzip2 对 `.tar` 文件进行压缩；
* tar 具有多种命令行参数风格
	* 现代 GNU 风格：`tar --extract --file <FILE>
	* UNIX 风格：`tar -xf <FILE>`
	* 传统风格：`tar xf <FILE>`
* 不同版本的 tar 具有不同的功能，某些版本能够自动检测出文件已压缩并自动解压，有的版本则需要使用命令行参数显式指定才能处理压缩包，还有的版本则完全不能处理压缩文件，需要先手动解压缩；
* 命令中的 `f` 参数若省略，tar 程序将等待用户输入；

### 构建

* `configure` 脚本通常由 autotools 自动生成；

# 创建分区

1. 通过 `lsblk` 找出用于安装 LFS 的磁盘，如 `sda`；
2. 建立新的 GPT
```bash
$ parted /dev/sda
(parted) mktable
New disk label type? gpt
Warning: The existing disk label on /dev/sda will be destroyed and all data on this disk will be lost. Do you want to continue?
Yes/No? Yes
(parted) quit
```
3. 使用 fdisk 或 cfdisk 进行分区创建，cfdisk 是带有 ncurses 界面的 fdisk
	1. `cfdisk /dev/sda` 进入分区创建程序；
	2. 先创建 EFI 分区，选中 `Free space` 再进行 `[New]` 操作，建议大小为 256 MB，接下来进行 `[Type]` 操作并选择 `EFI System`；
	3. 根据上述操作创建根分区，类型为默认 `Linux filesystem`；
	4. 最后进行 `[Write]` 操作，然后退出；
4. 格式化分区
	1. `mkfs.vfat /dev/sda1` 格式化 EFI 分区；
	2. `mkfs -v -t ext4 /dev/sda2` 格式化根分区；

# 挂载分区

* 所谓的“挂载分区”，实际上指的是挂载该分区中的文件系统，但由于一个分区中最多只包含一个文件系统，人们常不加区分地用“分区”指代分区中的文件系统；
* 挂载 LFS 文件系统
```bash
$ mkdir -pv $LFS
$ mount -v -t ext4 /dev/sda2 $LFS
```

# 获取软件包

1. 创建存放软件包的目录
```bash
$ mkdir -v $LFS/sources
# 为所有用户添加写权限和 sticky 标志，sticky 标志用于限制只有文件所有者才能删除文件
$ chmod -v a+wt $LFS/sources
```
2. 下载各版本手册附有的 `wget-list-systemd` 文件，并使用 wget 下载所有软件包和补丁
```bash
$ wget --input-file=wget-list-systemd --continue --directory-prefix=$LFS/sources
```
若网速不佳，可从[镜像站列表](https://www.linuxfromscratch.org/mirrors.html#files)中的镜像站中下载软件包归档；
3. 下载各版本手册附有的 `md5sums` 文件进行校验
```bash
$ mv md5sums $LFS/sources
$ pushd $LFS/sources
  md5sum -c md5sums
popd
```
4. 将文件所有者改为 root 用户，避免宿主系统使用的用户 UID 在 LFS 中未分配导致这些文件在 LFS 系统中属于一个没有命名的 UID
```bash
$ chown root:root $LFS/sources/*
```

# 创建目录

```bash
$ mkdir -pv $LFS/{etc,var} $LFS/usr/{bin,lib,sbin}
$ for i in bin lib sbin; do ln -sv usr/$i $LFS/$i; done
# 64 位系统
$ mkdir -pv $LFS/lib64
# 用于存放交叉编译工具的目录
$ mkdir -pv $LFS/tools
```
**注意：这里软链接使用的是相对路径，不能使用绝对路径！！！因为后续进入 chroot 环境后，路径不再是 $LFS 。**

# 配置环境

## 添加用于编译 LFS 的专用用户

1. 创建用户 lfs 和同名用户组
```bash
$ groupadd lfs
$ useradd -s /bin/bash -g lfs -m -k /dev/null lfs
$ passwd lfs
```
2. 将 $LFS 的所有目录的所有者设置为 lfs
```bash
$ chown -v lfs $LFS/{usr{,/*},lib,var,etc,bin,sbin,tools}
# 64 位系统
$ chown -v lfs $LFS/lib64
```
3. `su - lfs` 切换到 lfs 用户，`-` 参数表示启动一个登录 shell ；

## 配置工作环境

1. 新建 `.bash_profile`
```bash
$ cat > ~/.bash_profile << "EOF"
# 清空所有环境变量
exec env -i HOME=$HOME TERM=$TERM PS1='\u:\w\$ ' /bin/bash
EOF
```
2. 新建 `.bashrc`
```bash
cat > ~/.bashrc << "EOF"
set +h # 强制总是搜索 PATH 以访问最新程序
umask 022 # 设置默认文件创建掩码
LFS=/mnt/lfs
LC_ALL=POSIX
LFS_TGT=$(uname -m)-lfs-linux-gnu
PATH=/usr/bin
if [ ! -L /bin ]; then PATH=/bin:$PATH; fi
PATH=$LFS/tools/bin:$PATH
CONFIG_SITE=$LFS/usr/share/config.site
export LFS LC_ALL LFS_TGT PATH CONFIG_SITE
EOF
```
3. 若存在 `/etc/bash.bashrc`，建议将其移走避免影响工作环境；
4. 配置 `$MAKEFLAGS` 环境变量以指定并发任务数，**注意绝对不要使用不加数字的 `-j` 参数，将导致 make 生成无限多的任务**；
```bash
$ cat >> ~/.bashrc << "EOF"
export MAKEFLAGS=-j$(nproc)
EOF
```

# 交叉编译

* 常用术语
	* build 指构建程序时使用的机器
	* host 指构建出的产物将被运行的机器
	* target 指编译器产生的代码的目标机器
* 在俗称 Canadian Cross 的场景中，假设有 A、B、C 三台机器，其中只有 A 上有编译器 ccA，而希望在 B 上为 C 构建编译器，则可以分三步

| 阶段  | Build | Host | Target | 操作描述                    |
| :-: | :---: | :--: | :----: | :---------------------- |
|  1  |   A   |  A   |   A    | 在 A 上使用 ccA 构建交叉编译器 cc1 |
|  2  |   A   |  B   |   C    | 在 A 上使用 cc1 交叉编译 cc2    |
|  3  |   B   |  C   |   C    | 在 B 上使用 cc2 构建编译器 ccC   |
其中，cc1 和 cc2 是交叉编译器，为与它们本身所在的机器不同的机器产生代码，而 ccA 和 ccC 是本地编译器，为它们本身运行的机器产生代码；
* 基于 autoconf 的构建系统使用形如 **CPU - 供应商 - 内核 - 操作系统** 的**三元组**表示目标系统类型，其中供应商字段允许省略，如 `x84_64-unknown-linux-gnu`
	* 为什么三元组**有四个字段**？因为最开始没有内核和操作系统这两个字段，只有**系统**这一个字段，但是仅有这三个字段无法准确描述一些平台，如 Android 和基于 ARM64 架构的 Ubuntu 都拥有相同的 CPU 类型和内核，但是两者完全不同；
	* 许多软件包附带 `config.guess` 脚本可用于获取机器的三元组；使用 `gcc -dumpmachine` 也能获取类似的信息；
* 需要**注意动态链接器的名称**；在 32 位的 Intel 机器上通常为 `ld-linux.so.2`，而在 64 位系统中通常为 `ld-linux-x86_64.so.2`；可使用 `readelf -l <BINARY> | grep interpreter` 查看；
* 为了模拟交叉编译环境，在[配置工作环境](#配置环境#配置工作环境)一节中设置 `LFS_TGT` 环境变量为 `x86_64-lfs-linux-gnu`；
* `gcc -print-prog-name=ld` 查看 gcc 使用的链接器；

# 升级

* 升级 Linux 内核**不需要**重新构建任何软件包，也**不需要**一同更新 Linux API 头文件，重新引导系统即可；
* 如果更新了共享库，共享库的名称没有变化但是库文件的版本号**降低**了，例如 libfoo.so.1.25 降为 libfoo.so.1.24，则需要将旧库文件删除，否则 ldconfig 会将符号链接 libfoo.so.1 重新链接到旧库文件 libfoo.so.1.25 ，因为它的版本号更大；
* 如果更新了包含严重问题修复但名称没有改变的共享库，则需要重新启动所有链接到该库的程序，使用命令 `grep -l '<LIBRARY>.*deleted' /proc/*/maps | tr -cd 0-9\\n | xargs -r ps u` 列出所有正在使用旧版本共享库的进程；如果 systemd 也链接到该库，可以执行 `systemctl daemon-reexec` ；
* 如果可执行文件或共享库直接被覆盖，则可能导致正在使用该程序或库的代码或数据的进程崩溃；正确的做法是先删除旧版本，再安装新版本，这也是 `install` 命令所做的事情；