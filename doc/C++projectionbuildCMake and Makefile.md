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

意味着`Makefile`会在src和`../header`目录下进行文件搜索，也可以通过`vpath`关键字实现