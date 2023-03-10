# C++项目构建——CMake and Makefile

## Makefile

### 介绍

如何编译和链接源代码，规则是

1. 工程没有被编译过，编译所有C文件并连接
2. 工程中修改几个c文件，只编译修改的c文件，并重新连接
3. 头文件被修改，重新编译引用头文件的c文件

规则

```makefile
target ...: prerequisites ...
	command
	...
	...
```

1. target 目标文件/可执行文件/标签
2. prerequisites 生成target以来的文件
3. command 生成target执行的命令

> prerequisite包含一个文件比target要新，执行command命令

#### `Makefile`执行

执行`make`命令

1. 寻找Makefile文件
2. 寻找第一个目标文件，作为最终目标文件
3. 寻找依赖文件，不存在则寻找依赖文件的生成规则

#### 自动推导

`Make`命令对于$*.o$文件会自动将$*.c$文件家在依赖关系中，以下两个Makefile等价

```makefile
main: main.cpp head.o
	cc main.cpp head.o -o main
head.o: head.h
```

```makefile
main: main.cpp head.o
	cc main.cpp head.o -o main
head.o: head.h head.cpp
	cc head.cpp -c -o head.o
```

#### 清除目标文件

`make clean`用于删除编译生成的中间文件，推荐的写法是

```makefile
clean:
	-rm $(object)
```

避免遇到不存在的文件导致命令终止

#### What is in Makefile

1. 显式规则
2. 隐式规则
3. 定义变量
4. 文件指示 #include另一个Makefile或者定义多行命令
5. 注释

### Makefile规则

规则=依赖关系+生成目标的方法

#### 通配符

1. ~的使用 `~/test`表示home目录下的test子目录，`~chen/test`表示用户chen所属的home目录
2. `*`匹配任意字符 `?`匹配任意单个字符

用通配符只是简单的字符串代替，希望得到通配符对应的文件列表需要使用`wildcard`

#### 文件搜寻

头文件和目标文件一半放在不同的目录下，可以指定搜寻文件的目录

```makefile
VPATH = src:../header
```

意味着`Makefile`会在src和`../header`目录下进行文件搜索，也可以通过`vpath`关键字实现，比如

```
vpath %.h ../header
```

意味着对于所有模式为`%.h`的文件在`../header`下搜索

可以执行多条规则，如果一个文件名同时满足多项pattern，make按照vpath顺序选择

 ```makefile
vpath %.c src
vpath %.cpp src
 ```

> 内置变量
>
> 1. $@代表target
> 2. $<是第一个源
> 3. $^是所有的源
>
> command前加上@可以避免命令回显

#### 伪目标

有的目标并非是生成的目标文件，避免文件名和标签重复，可以使用

```makefile
.PHONY: clean
```

另一个例子是为伪目标指定文件依赖通过一个命令生成多个文件

```makefile
.PHONY: all
all: target1 target2 target3
target1: prerequisite1
	command
```

#### 多目标

`Makefile`规则中target可以包含不止一个目标

#### 自动生成依赖性

cpp中，依赖关系根据`include`的头文件决定，比如`main.cpp` `#include`了`def.h`文件，依赖关系是

```makefile
main.o: main.c defs.h
```

使用

```cpp
cc -M main.c
```

输出

```makefile
main.o: main.c defs.h
```

用gcc/g++需要使用-MM选项避免包含标准库依赖。建议为每个`.c`文件生成`.d`对应的`Makefile`文件，在`.d`文件中存放`.c`的依赖关系。

> 生成.d文件是为了让编译器的这个功能和`Makefile`结合在一起，如果include的头文件不是同一个子目录下执行-MM需要加上`-Iincludepath`选项

make自动更新/生成`.d`文件

```makefile
%.d: %.c
    @set -e; rm -f $@; \
    $(CC) -M $(CPPFLAGS) $< > $@.$$$$; \
    sed 's,\($*\)\.o[ :]*,\1.o $@ : ,g' < $@.$$$$ > $@; \
    rm -f $@.$$$$
```

1. 所有`.d`文件依赖`.c`文件，`rm -d $@`删除所有的目标
2. 每个依赖文件`.c`生成依赖文件 $@表示模式为`%.d`的文件

> sed命令按照脚本处理文本文件

### Makefile下的多目录编译

#### `Makefile include`

```makefile
include filename
```

挂起当前makefile内容的读取，先进入其它目录读取include指定的一个或多个`Makefile`，然后处理当前的makefile

> 多个目录下文件编译由各个makefile分布式处理，使用同一组变量和模式规则

多个makefile可以对同一个变量操作，比如一个目录下

#### 指定搜索目录

假设目录中存在如下文件

```cpp
main.c foo.c foo.h
```

编译`main`可执行文件不需要编译foo.h

```shell
cc main.c foo.c -o main
```

因为.h文件会作为头文件整体添加到`main.c`中

多目录下进行文件搜索需要指定include目录，需要添加`-I`选项或者`--include-dir`，通过vpath添加对应目录可以处理**目标依赖**中的文件自动目录，比如

```makefile
VPATH += src
all:foo.c
	cc $^ -o $@
```

等效于

```makefile
cc src/foo.c -o all
```

不能直接在command中调用`foo.c`，会导致文件找不到

### 命令

#### 显示命令

makefile默认回显每条command，希望不要回显需要在命令行前加上@

```makefile
@echo compile
```

#### 命令执行

依赖比目标更新，make会执行所有命令，**命令之间存在依赖关系则需要使用;分割并且需要在用一行**

#### 命令出错

在命令前加上-避免命令出错

#### 嵌套执行make

子目录subdir下存在makefile用于编译子目录下文件的编译过程，父目录下编译子目录下对象写成

```makefile
subsystem:
	cd subdir; $(MAKE)
```

等价于

```makefile
subsystem:
	$(MAKE) -C subdir
```

传递变量到下级makefile中，可以使用

```makefile
export <variablelist>
```

比如

```makefile
export variable = value
```

等价于

```makefile
variable = value
export variable
```

或者

```makefile
export variable := value
```

## `Makefile`变量和自动化变量

`Makefile`定义的变量有点类似MACRO

### 变量基础

使用=定义变量，可以用其它变量定义变量

```makefile
goo=$(bar)
bar=hello
all:
	echo $(bar)
```

> 定义`goo`并未定义`bar`的值，可以借助后面的值缺省定义，这样可以根据真实值推断不同的变量。需要避免递归定义

调用变量需要用$，例如`$(varname)`，变量会在调用处精确展开为字符串序列

#### 立即赋值和延迟赋值

使用`:=`定义变量

```makefile
x := foo
y := $(x) bar
x := later
```

此时y的值是`foo bar`而不是`later bar`，有点类似编译时求值和运行时求值的区别

makefile提供了一些系统变量，比如MAKELEVEL是makefile执行的层次

定义变量控制是否包含空格可以借助#，比如

```makefile
dir := /foo/bar #command
```

则dir带一个空格

```makefile
dir := /foo/bar#command
```

不包含空格

操作符`?=`功能是

1. 如果变量未定义，则定义变量并采用右值
2. 变量已经定义，则为空语句

`+=`可以给变量追加值，相当于字符串连接

### 自动化变量

| 变量名 | 含义                           |
| ------ | ------------------------------ |
| $*     | 不包括扩展名的**目标文件**名称 |
| $+     | 所有的依赖文件，以空格分隔     |
| $<     | 第一个依赖文件                 |
| $?     | 比目标更新的依赖文件           |
| $@     | 目标文件完整名称               |
| $^     | 不重复的依赖文件               |

## 调用函数和条件判断

### 条件判断

语法是

```makefile
<conditional-directive>
<text-if-true>
endif
```

或者

```makefile
<conditional-directive>
<text-if-true>
else
<text-if-false>
endif
```

`conditional-directive`是条件关键字，包括

```makefile
ifeq (arg1,arg2)
ifdef <variable name> # 测试变量是否有值，如果未赋值则不执行，不如idef foo
ifneq (arg1,arg2) # 不相等则执行
ifndef <variable name> # 变量值未定义则执行
```

