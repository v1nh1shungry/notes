**原文链接：https://fabiensanglard.net/dc/index.php**

# 驱动程序

* 平常使用的 `gcc` 、 `clang` 等命令只是编译器驱动程序，它会依次调用**真正的编译器**、汇编器和链接器，使用 `-v` 可以查看驱动程序调用的程序及参数， `-###` 则能只打印驱动程序将调用的程序而不实际执行；
* `readlink` 可查看链接到的源程序， `readlink $(which cc)` 可知 `cc` 是 `gcc` 的别名；
* 驱动程序将自动识别参数并路由至对应的程序，如 `-std=c++20` 将路由至编译器，而 `-lm` 将路由至链接器。**可以使用 `-Wl` 显示将参数转发给链接器；**
* 驱动器会根据**扩展名**自动检测源文件所用的语言，并通过 `-x` 参数传递给编译器指定所用的编程语言；

# 预处理器

* 最开始预处理器 `cpp` 作为一个独立可执行程序使用，后来其被合并入编译器中，比起将源文件通过独立的预处理器产生的结果写入磁盘再供编译器使用，直接在内存中处理能有效减少 IO 次数，早年间的做法更像是早期计算机内存不足的妥协；
* `cpp -### main.cpp` 能看出现在的 `cpp` 命令将调用 `gcc` 或 `clang`；
* 在预处理结果中，每个代码片段前都有注释 `# linenum filename` 用于编译器能追溯代码来自哪个文件的哪一行，从而能提供精确的报错信息；
* 头文件的缺陷是在大项目中很容易成为编译速度瓶颈。头文件很可能被多个编译单元包含，每个编译单元在编译时都要解析一次头文件；头文件很可能包含了若干个其他头文件，这使得头文件在经过预处理展开后会变得巨大，这进一步增加了编译工作；随着 `header-only` 库的流行，同样放大了头文件编译速度慢的问题；
	* 大项目中尽可能将头文件中对其他头文件的依赖减小，尽可能使用前向声明，既能减小预处理结果也能减少头文件变动影响的重编译范围；尽量将头文件中的内容转移至单独的编译单元（反 `header-only`），更加极端的做法是使用 `PImpl` 模式；
	* 使用预编译头。`clang -cc1 pch.h -emit-pch -o pch.pch` 编译预编译头，源文件将不必再包含头文件，在编译源文件时将预编译头传递给编译器 `clang -include-pch pch.pch -c source.c`；
* `-sysroot` 、 `-iquote` 、 `-isystem` 将影响 `#include` 使用 `<>` 还是 `""` ，此外，`-sysroot` 能指定一个完整的工具链，包括头文件搜索路径和库链接器搜索路径；
* 某些头文件由编译器提供，它们将自动包含在头文件的搜索路径中，如 `stddef.h`；
* `-MD` 使预处理器输出编译单元的依赖关系，`-MF` 将指定输出的 `.d` 文件名，[[跟我一起写 Makefile#自动生成依赖|这些依赖关系可供 `make` 等构建系统使用]]；

# `binutils`

* `binutils` 列表
```
Name      |  Description
--------------------------------------------------------------
ld        |  The GNU linker.
as        |  The GNU assembler.
--------------------------------------------------------------
addr2line |  Converts addresses into filenames and line numbers.
ar        |  Creates, modifies and extracts from archives.
c++filt   |  Filter to demangle encoded C++ symbols.
dlltool   |  Creates files for building and using DLLs.
gold      |  New, faster, ELF only linker, 5x faster than ld.
gprof     |  Displays profiling information.
ldd       |  List libraries imported by object file.
nlmconv   |  Converts object code into an NLM.
nm        |  Lists symbols from object files.
objcopy   |  Copies and translates object files.
objdump   |  Displays information from object files.
ranlib    |  Generates an index to the contents of an archive.
readelf   |  Displays information from any ELF format object file.
size      |  Lists the section sizes of an object or archive file.
strings   |  Lists printable strings from files.
strip     |  Discards symbols.
windmc    |  Windows compatible message compiler.
windres   |  Compiler for Windows resource files.
```
* 利用 `readelf` 读取程序入口地址，再使用 `nm` 找到入口符号
```bash
$ readelf -h main | grep Entry
  Entry point address:               0x1080
$ nm main | grep 1080
0000000000001080 T _start
```