---
layout: post
title: make和Makefile，多文件项目管理
categories: 好资源之开发神器
tags: 开发工具 make Makefile linux
---

# GNU Make简介

大型项目的开发过程中，往往会划分出若干个功能模块，这样可以保证软件的易维护性。

作为项目的组成部分，各个模块不可避免的存在各种联系，如果其中某个模块发生改动，那么其他的模块需要相应的更新。如果通过手动去完成这个工作的话，对于小型的项目可能还行，但是对于比较大型的项目就几乎是不可能的。

因此Linux 系统提供了一个自动维护和生成目标程序的工具 make，它可以根据各个模块的更改情况去重新编译连接目标代码。

Make 工具的作用就是实现编译连接过程的自动化。它定义了一种语言，用来描述源文件、目标文件以及可执行文件之间的关系，通过检查文件的时间戳来决定程序中哪些文件需要更新编译，并发送相应的命令。

我们在开发项目的时候，将程序划分为多个模块，分解到不同的文件中之后。当其中的某一部分发生改变之后，因为其他文件的目标源文件已经存在，所以编译器其实不需要编译全部代码来生成新的可执行文件，而只需要编译被改动的源文件，然后连接所有的目标文件就可以了，这在大型的项目开发中是非常重要的，因为这可能将编译时间从几小时缩小到几分钟。这就是Make所能做的。

# Makefile文件书写规范

Makefile 文件描述了整个程序的编译、连接规则，主要包括：程序中哪些源文件需要编译以及如何编译，需要创建哪些库文件以及如何创建这些文件，如何产生最终的可执行文件等。

### 基本规则

`#` 开始的行是注释行。

如果一行太长可以用 反斜线 `\` 来另起一行，相当于就是一行。

Makefile文件的作用是告诉 make工具做什么，多数情况下是如何编译连接一个程序：

```makefile
目标 : 依赖
<tab键>命令
```

`目标`，往往是程序的中间或者最终生成的文件名，比如目标文件、可执行文件……

`依赖`，是指用来产生目标文件的输入文件名，一个目标往往依赖于一个或多个文件。

`命令`，是指任何一个文件发生改动之后，需要重新生成目标文件需要执行的命令，这里可以有多条命令，但是每个命令必须单独占一行，且需要注意的是，每个命令的前面必须有一个<tab键>，因为make是用过<tab>来识别命令行的，进而完成相应的动作。

**例子1**，首先是一个简单的 hello.c的C语言源程序：

```c
#include<stdio.h>
int main()
{
    printf("hello world\n");
    return 0;
}
```

它的Makefile文件可以是：

```makefile
/**
目标 : 依赖
<tab键>命令
**/
hello:hello.c
    gcc -o hello hello.c
```

然后在Makefile和hello.c 所在的目录下执行 make 命令，就可以编译hello.c 生成 hello可执行文件。

**例子2**，上面那个例子太简单，只有一个文件，下面的这个例子有三个文件和两个头文件。

大致的过程如图：

![img](../media/image/2015-06-05/make.png)

它的对应的Makefile 文件是：

```makefile
example:sort.o compute.o main.o
    gcc sort.o compute.o main.o -o example
sort.o:sort.c lib1.h
    gcc -c sort.c -o sort.o
compute.o:compute.c
    gcc -c compute.c -o compute.o
main.o:main.c lib2.h
    gcc -c main.c -o mian.o
```

注意最后需要生成的文件，需要在Makefile 里面写在最前面，它的大致的执行的逻辑是这样的：

1) 在命令行输入 make 命令之后，make命令会首先读取当前目录下的 Makefile文件

2) make 会处理 Makefile里面的第一条规则，也就是上面的 连接生成 example可执行文件

3) 完成第一条规则之后，需要首先处理的是 example所依赖的目标文件，也就是 sort.o、compute.o、main.o，目标文件根据其依赖的源文件或者头文件是否比现在的目标文件更新，或者是目标文件是否存在来决定是否需要重新编译。目标文件处理完成之后，make才会决定是否需要重新连接生成可执行文件，存在三种情况：

1. 如果所有的文件都没有被编译，则编译所有的源文件，并连接生成可执行文件
2. 重新编译上次执行 make 命令之后修改过的源文件，生成新的目标文件，然后和已经存在的，但这次没有编译的文件重新连接生成可执行文件
3. 如果某个头文件在上次执行 make 命令之后被修改，则重新编译所有包含这个头文件的源文件，生成新的目标文件，然后和已经存在的，但这次没有编译的文件重新连接生成可执行文件

### 变量的定义和使用

Makefile 里面可以定义一个变量来代替一个字符串，这些字符串可以是目标、依赖或者是命令，以及Makefile 的其他部分，引用变量的值的时候，只需要使用 $ 就好，例子：

```makefile
objects=sort.o compute.o main.o
CC=gcc
CFLAGS=-Wall -g
example:$(objects)
    $(CC) &(objects) -o example
sort.o:sort.c lib1.h
    $(CC) $(CFLAGS) -c sort.c -o sort.o
compute.o:compute.c
    $(CC) $(CFLAGS) -c compute.c -o compute.o
mian.o:mian.c lib2.h
    $(CC) $(FLAGS) -c mian.c -o mian.o
```

变量名是大小写敏感度的，一般命令相关的变量习惯用大写（CC），文件相关的变量习惯用小写（objects），参数相关的变量也习惯使用大写（CFLAGS）。

如果变量名是单字符，可以直接使用 $变量名来引用，$C；但是变量名为多于一个字符的字符串，在引用的时候，必须使用$(变量名) 的形式，比如 $(CFLAGS)，否则make工具只会解析第一个字符，比如 $CFLAGS，将只会解析 $C 的变量，后面的FLAGS作为普通的字符串看待，等价于$(C)FLAGS，所以就可能出错。

根据变量的定义和展方式是不同，可以将Makefile里面的变量分为：

**1) 递归展开式变量**

通过 = 来进行定义，引用的时候进行严格的文本替换，变量中对于其他的变量或者函数的引用在使用时候才进行展开。例子：

```makefile
A=$(B)
B=$(C)
C=Hello
```

如果在这个Makefile 里面存在对变量 A的引用：$(A)，那么在执行make 命令的时候，变量开始替换，首先将变量 A替换为变量 B，接下来替换为变量C， 最终替换为 Hello。

递归展开式的优点是，变量定义的时候可以引用后续定义的变量。

缺点是，有可能在变量展开时出现无穷的循环，这就很蛋疼了。

**2) 直接展开式变量**

为了避免递归展开式变量存在的问题，所以可以使用直接展开式变量，通过:= 进行定义。变量中对于其他的变量或者函数的引用在定义时候就进行展开。

例子1

```makefile
A=Hello
B:=$(A)World
A:=HI
```

因为是在定义的时候就展开，所以，变量B 的值是HelloWorld，而不像递归展开式中会是HIWorld（递归展开式变量中，A会持续对B造成影响），因为A首先定义为 Hello，然后定义B，因为在定义时候就展开，所以B的值是HelloWorld，而后面再重新定义A 的话是不会对B在造成影响的。

例子2

```makefile
B:=$(A)World
A=Hi
```

则最终 B的值是 World，因为在定义B 的时候A 还没有定义，所以make 会认为A 是空。

### 隐含规则

隐含规则是系统或用户预先定义好的一些特殊规则，主要是一些常用的依赖关系和更新命令。

一般规则使用文件的全名，而隐含规则中出现的目标文件和依赖文件都只使用文件的扩展名。如果Makefile 文件里面没有显式给出文件的依赖关系的时候，make 就会根据文件的扩展名找到相应的隐含规则，然后按照隐含规则来更新目标。

例子，隐含规则是：

```makefile
.c:
    $(CC) $(CFLAGS) -o &@ $<
.c .o:
    $(CC) $(CFLAGS) -c $<
```

下面给出的Makefile就是使用上面的隐含规则：

```makefile
objects=sort.o compute.o mian.o
CC=gcc
CFLAGS=-Wall -g
example:$(objects)
    $(CC) $^ -o $@
sort.o:lib1.h
mian.o:lib2.h
```

### 伪目标

Makefile 文件中的目标分为两类：实目标和伪目标。

实目标是真正要生成的以文件形式存放在磁盘上的目标，上面所讲解到的都属于实目标；而伪目标不要求生成实际的文件，它主要是用于完成一些辅助操作。例子：

```makefile
clean
    rm example $(objects)
```

在Makefile 里面增加了上面的规则之后，在命令里面输入命令：make clean 就会执行命令：`rm example sort.o compute.o mian.o`

但是这种书写形式不是很严谨，因为可能在当前目录下面存在文件名为 clean 的文件，因为这时候: 后面没有依赖文件，所以make 就认为这个文件是最新的，所以就不会执行 rm example sort.o compute.o mian.o

所以为了避免这种情况的发生，所以建议使用这种：

```makefile
.PHONY:clean
clean:
    rm example $(objects)
```

这样，不管当前目录下是否存在文件名为 clean 的文件，rm example sort.o compute.o mian.o命令都会被执行。

### 函数

GNU make提供了很多的函数，可以在Makefile文件中调用这些函数来进行文件名、变量以及命令等的处理。

函数的调用方式与变量类似，使用 $ 符号。

**1) patsubst 函数**

主要用于对字符串经行运算和分析，格式是：

```makefile
$(patsubst pattern,replacement,text)
```

例子1

```makefile
$(patsubst %.c,%.o,sort.o compute.c main.c)
```

这个就是输出与源文件相对应的目标文件列表，输出为：

```makefile
sort.o compute.o main.o
```

**2) dir 函数**

主要用于获取文件的路径，例子：

```makefile
$(dir main.c)
```

如果main.c 在当前目录下，就会输出：

```makefile
./　　//使用相对路径的形式
```

**3) notdir 函数**

抽取文件名中除了路径之外的其他字符，例子：

```makefile
$(notdir /home/perfect/Mywork/C/main.c ./Makefile
```

输出是：

```makefile
main.c Makefile
```

**4) suffix 函数**

获取文件名的后缀：

```makefile
$(suffix ./main.c)
```

输出结果是：

```makefile
.c　　//也就是 ./main.c上的后缀
```

通用Makefile文件
========

可以看出，编写一个Makefile还是很复杂的。

下面给出一个通用的Makefile 文件，其作者是应该的Gorge Foot，之所以说它是通用的，主要是因为它不需要经过修改就可以应用于大部分的项目之中：

```makefile
######################################
# Copyright (c) 1997 George Foot (george.foot@merton.ox.ac.uk)
# All rights reserved.
######################################
#目标（可执行文档）名称，库（譬如stdcx,iostr,mysql等），头文件路径
DESTINATION := test
LIBS := 
INCLUDES := .


RM := rm -f
#C,CC或CPP文件的后缀
PS=cpp
# GNU Make的隐含变量定义
CC=g++
CPPFLAGS = -g -Wall -O3 -march=i486
CPPFLAGS += $(addprefix -I,$(INCLUDES))
CPPFLAGS += -MMD

#以下部分无需修改
SOURCE := $(wildcard *.$(PS))
OBJS := $(patsubst %.$(PS),%.o,$(SOURCE))
DEPS := $(patsubst %.o,%.d,$(OBJS))
MISSING_DEPS := $(filter-out $(wildcard $(DEPS)),$(DEPS))
MISSING_DEPS_SOURCES := $(wildcard $(patsubst %.d,%.$(PS),$(MISSING_DEPS)))

.PHONY : all deps objs clean rebuild

all : $(DESTINATION)

deps : $(DEPS)
         $(CC) -MM -MMD $(SOURCE)

objs : $(OBJS)

clean :
         @$(RM) *.o
         @$(RM) *.d
         @$(RM) $(DESTINATION)

rebuild: clean all 

ifneq ($(MISSING_DEPS),)
$(MISSING_DEPS) :
         @$(RM) $(patsubst %.d,%.o,$@)
endif

-include $(DEPS)

$(DESTINATION) : $(OBJS)
         $(CC) -o $(DESTINATION) $(OBJS) $(addprefix -l,$(LIBS))
#结束
```

简介（通过研究这个Makefile 文件可以很好的理解Makefile的规则）：

- 原作者是Gorge Foot，写这个Makefile的时候还是一个学生
- ":="赋值，和"="不同的是，":="在赋值的同时，会将赋值语句中所有的变量就地展开，也就是说，A:=$(B)后，B的值的改变不再影响A
- 隐含规则。GUN Make在不特别指定的情况下会使用诸如以下编译命令：$(CC) $(CFLAGS) $(CPPFLAGS) $(TARGET_ARCH) -c $< -o $@，这也是为什么这个Makefile最后一个命令没有添加$(CPPFLAGS)的原因，因为缺省是包含这个变量的
- 函数和变量很相似："$ (函数名，空格，一列由逗号分隔的参数)"
- SOURCES = $(wildcard \*.cpp) 列出工作目录下文件名满足"\*.cpp"条件的文件，以空格分隔，并将列表赋给SOURCE变量
- patsubst函数：3个参数。功能是将第三个参数中的每一项（由空格分隔）符合第一个参数描述的部分替换成第二个参数制定的值
- addprefix函数：2个参数。将源串（第2个参数，由空格分隔）中的每一项添加前缀（第1个参数）
- filter-out函数：2个参数。从第二串中过滤掉包含在第一个串中的项
- $(CC) -MM -MMD $(SOURCE) : 对每个源文件生成依赖(dependence，Make通过依赖规则来判断是否需要重新编译某个文件)，"D"生成".d"文件，-MM表示去掉 depends里面的系统的头文件(使用<>包含的头文件)（若使用-M则全部包含，事实上，系统头文件被修改的可能性极小，不需要执行依赖检查）
- .PHONY，不检查后面制定各项是否存在同名文件
- ifneg...else...endif，Makefile中的条件语句
- -include $(DEPS) : 将DEPS中的文件包含进来，"-"表示忽略文件不存在的错误
- @$(RM) \*.o : 开头的"@"表示在Make的时候，不显示这条命令（GNU Make缺省是显示的)
- all : 作为第一个出现的目标项目，Make会将它作为主要和缺省项目("make"就表示"make all")
- deps : 只生成依赖文件(.d文件)
- objs : 为每一个源码程序生成或更新 '.d' 文件和'.o'文件
- clean : 删除所有'.d'、'.o'和可执行文件
- rebuild : clean然后重建
- 内部变量$@、$<、$^ : 分别表示目标名（:前面的部分，比如all），依靠列表（:后面的部分）中的第一个依靠文件，所有依靠文件

