# 自动推导

* 自动推导源文件依赖、指令
```make
playground: playground.o
```
可自动推导出依赖的源文件和使用的编译指令
```make
playground: playground.o
  $(CC) -o playground playground.o $(LDFLAGS) $(LOADLIBES) $(LDLIBS)

main.o: main.c
  $(CC) -c main.c $(CPPFLAGS) $(CFLAGS)
```
**注意：`CFLAGS` 和 `LDFLAGS` 等编译参数会自动加入推导出的隐式规则中。**
* 通过写出隐式规则而不写命令以禁用该隐式规则，如
```make
%.o: %.c
```
此时 `make` 不再为 `.o` 文件自动推导出构建规则。

# 命令行参数

* 尽量不要使用 `-I` 命令行参数来指定 `include` 目录，因为会覆盖已经设定的包含目录，包括默认包含目录。
* `-n` 只显示命令不执行（dry-run），`-s` 将禁止显示命令。

# 文件搜索

* 使用 `VPATH` 变量指定**源文件**搜索目录，可指定多个路径，使用 `:` 分隔（与 `bash` 类似）。
* 使用 `vpath` 命令（推荐）。
```make
# 在以下路径搜索 C 源文件
# 搜索顺序：foo/, bar/, foobar/
vpath %.c foo:bar
vpath %.c foobar
```

# 自动生成依赖

* 编译器生成依赖关系
```bash
# 为 main.c 生成依赖关系
gcc -M main.c
# 不包含系统头文件
gcc -MM main.c
```
* 使用 `.d` 文件自动更新依赖关系
```make
sources = foo.c bar.c

# 生成 .d 文件，并往编译器生成的依赖关系中加入生成的 .d 文件
%.d: %.c
  @set -e; rm -f $@; \
  $(CC) -MM $< > $@.$$$$; \
  sed 's,\($*\).o[ :]*,\1.o $@ : ,g' < $@.$$$$ > $@; \
  rm -f $@.$$$$

# 将 sources 中所有的 .c 替换为 .d
include $(sources:.c=.d)
```
`.d` 文件依赖对应的 `.c` 文件，当源文件发生改动时，自动调用编译器生成新的依赖关系，同时，将 `.d` 文件添加到生成的依赖关系中，当依赖的文件发生改动时，也能自动更新 `.d` 文件。

# 命令执行

* 使用 `cd` 时，应将需要 `cd` 后执行的命令放在与 `cd` 指令同行
```make
clean:
  cd build/; rm *.o
```
`make` 的每条指令应该都是单独开一条子进程执行的，所以上一条 `cd` 命令在下一条指令不会生效。
* 忽略错误的三种途径
	* 命令前添加 `-`。
	* 使用命令行参数 `-i` 忽略一切错误；`-k` 出错时终止当前规则，但继续执行其他规则。
	* `.IGNORE` 伪目标。

# 嵌套 `make`

* 使用 `$(MAKE)` 为 `make` 指定通用的命令行参数，或设置 `MAKEFLAGS` 变量，该变量将默认传递给子 `make` 实例。
* 使用 `export` 将变量传递给子 `make` 实例。

# 变量

* 使用 `x = $(y)` 形式定义时，可引用后面定义的变量；使用 `x := $(y)` 形式定义时，则不能引用后面定义的变量，**注意：当使用当前还未未定义的变量时，将展开为空而不会报错**。
* 在定义变量时，使用行注释需要注意，到行注释的空格也是包含在变量定义当中的。
```make
nullstring :=
space := $(nullstring) # end of the line
```
这里先定义了一个空变量 `nullstring`，再在定义 `space` 时利用 `$(nullstring)` 和行注释之间的空格，将 `space` 的值定义为一个空格，非常变态的定义空格的方法，`make` 真的是……
* 使用 `override` 来修改由命令行参数定义的变量。
* 使用目标/模式变量来覆盖全局变量或环境变量
```make
# 无论 CFLAGS 变量为何值，在 playground 规则内该变量的值都为 -g
playground: CFLAGS = -g
playground: main.o
  $(CC) $(CFLAGS) main.o -o playground

# 所有 .o 文件的规则中，CFLAGS 的值都为 -g
%.o: CFLAGS = -g
```

# 函数

* 自动生成 `include` 路径
```make
override CFLAGS += $(pubsubst %,-I%,$(subst :, ,$(VPATH)))
```

# 并行

* 打包归档文件时，并行构建**可能有多个 `ar` 命令在同时操作同一个归档文件，造成归档文件损坏**。