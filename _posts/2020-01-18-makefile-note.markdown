---
layout:     post
title:      "Makefile"
subtitle:   "GNU make"
date:       2019-11-06
author:     "Dylan"
catalog: true
tags:
    - Makefile
    - Notes
---



> 摘录自 [跟我一起写 Makefile](https://blog.csdn.net/haoel/article/details/2886)


## 程序的编译和链接

把源文件编译成中间代码文件，即 Object File，这个动作叫做编译(compile). 然后再把大量的Object File合成执行文件，这个动作叫作链接(link). 编译时，编译器只检测程序语法，和函数、变量是否被声明。如果函数未被声明，编译器会给出一个警告，但可以生成Object File。而在链接程序时，链接器会在所有的Object File中找寻函数的实现，如果找不到，那到就会报链接错误码(Linker Error)

由于源文件太多，编译生成的中间目标文件太多，而在链接时需要明显地指出中间目标文件名，这对于编译很不方便，所以，我们要给中间目标文件打个包，在Windows下这种包叫“库文件”（Library File)，也就是 .lib 文件，在UNIX下，是Archive File，也就是 .a 文件


## Makefile


#### Makefile 规则

* 如果这个工程没有编译过，那么我们的所有C文件都要编译并被链接

* 如果这个工程的某几个C文件被修改，那么我们只编译被修改的C文件，并链接目标程序

* 如果这个工程的头文件被改变了，那么我们需要编译引用了这几个头文件的C文件，并链接目标程序

make会一层又一层地去找文件的依赖关系，直到最终编译出第一个目标文件。在找寻的过程中，如果出现错误，比如最后被依赖的文件找不到，那么make就会直接退出，并报错，而对于所定义的命令的错误，或是编译不成功，make根本不理。make只管文件的依赖性

Makefile中只有行注释, 用“#”字符

在命令行之间中的空格或是空行会被忽略，但是如果该空格或空行是以Tab键开头的，那么make会认为其是一个空命令

#### 语法

```
targets : prerequisites
    command
    ...

或者

targets : prerequisites ; command
    command
    ...
```

targets是文件名，一个或多个，以空格分开，可以使用通配符

command是命令行，单独一行必须以[Tab键]开头，如果和prerequisites在一行，那么可以用分号做为分隔

prerequisites是目标所依赖的文件(或依赖目标)

如果命令太长，你可以使用反斜框（‘/’）作为换行符

一般来说，make会以UNIX的标准Shell，也就是/bin/sh来执行命令

#### GNU make的工作步骤

* 读入所有的Makefile

* 读入被include的其它Makefile

* 初始化文件中的变量

* 推导隐晦规则，并分析所有规则

* 为所有的目标文件创建依赖关系链

* 根据依赖关系，决定哪些目标要重新生成

* 执行生成命令


#### 示例

1、基本写法

```
edit : main.o kbd.o command.o display.o /
       insert.o search.o files.o utils.o
    cc -o edit main.o kbd.o command.o display.o /
       insert.o search.o files.o utils.o
main.o : main.c defs.h
    cc -c main.c
kbd.o : kbd.c defs.h command.h
    cc -c kbd.c
command.o : command.c defs.h command.h
    cc -c command.c
display.o : display.c defs.h buffer.h
    cc -c display.c
insert.o : insert.c defs.h buffer.h
    cc -c insert.c
search.o : search.c defs.h buffer.h
    cc -c search.c
files.o : files.c defs.h buffer.h command.h
    cc -c files.c
utils.o : utils.c defs.h
    cc -c utils.c

clean :
    rm edit main.o kbd.o command.o display.o /
    insert.o search.o files.o utils.o
```

2、使用变量

```
objects = main.o kbd.o command.o display.o /
          insert.o search.o files.o utils.o

edit : $(objects)
    cc -o edit $(objects)
main.o : main.c defs.h
    cc -c main.c
kbd.o : kbd.c defs.h command.h
    cc -c kbd.c
command.o : command.c defs.h command.h
    cc -c command.c
display.o : display.c defs.h buffer.h
    cc -c display.c
insert.o : insert.c defs.h buffer.h
    cc -c insert.c
search.o : search.c defs.h buffer.h
    cc -c search.c
files.o : files.c defs.h buffer.h command.h
    cc -c files.c
utils.o : utils.c defs.h
    cc -c utils.c

clean :
    rm edit $(objects)
```

3、使用自动推导

```
objects = main.o kbd.o command.o display.o /
          insert.o search.o files.o utils.o

edit : $(objects)
    cc -o edit $(objects)

main.o : defs.h
kbd.o : defs.h command.h
command.o : defs.h command.h
display.o : defs.h buffer.h
insert.o : defs.h buffer.h
search.o : defs.h buffer.h
files.o : defs.h buffer.h command.h
utils.o : defs.h

.PHONY : clean
clean :
    rm edit $(objects)
```

4、其他风格

```
objects = main.o kbd.o command.o display.o /
          insert.o search.o files.o utils.o

edit : $(objects)
    cc -o edit $(objects)

$(objects) : defs.h
kbd.o command.o files.o : command.h
display.o insert.o search.o files.o : buffer.h

.PHONY : clean
clean :
    rm edit $(objects)
```


## 特性


#### 引用其它的Makefile

`include <filename>`

被包含的文件会原模原样的放在当前文件的包含位置

在include前面可以有一些空字符，但是绝不能是[Tab]键开始。include和<filename>可以用一个或多个空格隔开

eg: `include foo.make *.mk $(bar)`


#### 在规则中使用通配符


#### 文件搜寻

特殊变量“VPATH”, 由“:”分隔, 并按序搜索. 定义了这个变量，make就会在当前目录找不到的情况下，到所指定的目录中去找寻文件. 如果没有指明这个变量，make只会在当前的目录中去找寻依赖文件和目标文件

eg: `VPATH = src:../headers`

还可以用make关键字vpath


#### 伪目标

“伪目标”并不是一个文件，只是一个标签，由于“伪目标”不是文件，所以make无法生成它的依赖关系和决定它是否要执行。我们只有通过显示地指明这个“目标”才能让其生效。当然，“伪目标”的取名不能和文件名重名，为了避免和文件重名的这种情况，我们可以使用一个特殊的标记“.PHONY”来显示地指明一个目标是“伪目标”，向make说明，不管是否有这个文件，这个目标就是“伪目标”. 只要有这个声明，不管是否有“clean”文件，要运行“clean”这个目标，只有“make clean”这样

也可以为伪目标指定所依赖的文件。伪目标同样可以作为“默认目标”，只要将其放在第一个. 一个示例就是，如果你的Makefile需要一口气生成若干个可执行文件，但你只想简单地敲一个make完事，并且，所有的目标文件都写在一个Makefile中，那么你可以使用“伪目标”这个特性：

```
all : prog1 prog2 prog3
.PHONY : all

prog1 : prog1.o utils.o
    cc -o prog1 prog1.o utils.o

prog2 : prog2.o
    cc -o prog2 prog2.o

prog3 : prog3.o sort.o utils.o
    cc -o prog3 prog3.o sort.o utils.o
```

从上面的例子我们可以看出，目标也可以成为依赖。所以，伪目标同样也可成为依赖。看下面的例子：

```
.PHONY: cleanall cleanobj cleandiff

cleanall : cleanobj cleandiff
    rm program

cleanobj :
    rm *.o

cleandiff :
    rm *.diff
```

可以输入“make cleanall”和“make cleanobj”和“make cleandiff”命令来达到清除不同种类文件的目的


#### 多目标

有可能我们的多个目标同时依赖于一个文件，并且其生成的命令大体类似。于是我们就能把其合并起来

自动化变量“$@”表示目前规则中所有的目标的集合

```
bigoutput littleoutput : text.g
    generate text.g -$(subst output,,$@) > $@
```

等价于

```
bigoutput : text.g
    generate text.g -big > bigoutput
littleoutput : text.g
    generate text.g -little > littleoutput
```

-$(subst output,,$@)中的“$”表示执行一个Makefile的函数，函数名为subst，后面的为参数。这个函数是截取字符串的意思，“$@”表示目标的集合，就像一个数组，“$@”依次取出目标，并执行命令


#### 自动生成依赖性


#### 嵌套执行make

1、在每个目录中都书写一个该目录的Makefile，这有利于让我们的Makefile变得更加地简洁

2、可以用`cd subdir && make`或`make -C subdir`来使用嵌套make

3、上层Makefile的变量可以传递到下级的Makefile中(`export <variable ...>` 或 `export` #传递全部)，但是不会覆盖下层的Makefile中所定义的变量，除非指定了“-e”参数

4、有两个变量，一个是SHELL，一个是MAKEFLAGS，这两个变量不管你是否export，其总是要传递到下层Makefile中，特别是MAKEFILES变量，其中包含了make的参数信息，如果我们执行“总控Makefile”时有make参数或是在上层Makefile中定义了这个变量，那么MAKEFILES变量将会是这些参数，并会传递到下层Makefile中，这是一个系统级的环境变量

5、make命令中的有几个参数并不往下传递，它们是“-C”,“-f”,“-h”“-o”和“-W”

6、使用“-C”参数来指定make下层Makefile时，“-w”会被自动打开的。如果参数中有“-s”（“--slient”）或是“--no-print-directory”，那么，“-w”总是失效的


#### 定义命令包

可以为这些相同的命令序列定义一个变量, 命令包中的每个命令会被依次独立执行. 定义这种命令序列的语法以“define”开始，以“endef”结束, eg:

```
define run-yacc
yacc $(firstword $^)
mv y.tab.c $@
endef
```

```
foo.c : foo.y
    $(run-yacc)
```


## 命令

1、当我们用“@”字符在命令行前，那么，这个命令将不被make显示出来. "@echo 正在编译XXX模块......"执行时会输出“正在编译XXX模块......”字串，但不会输出命令

2、参数“-n”或“--just-print”，那么其只是显示命令，但不会执行命令. 参数“-s”或“--slient”则是全面禁止命令的显示

3、make会一条一条的执行其后的命令，但如果你要让上一条命令的结果应用在下一条命令时，你应该使用分号分隔这两条命令，而不是把这两条命令写在两行上，eg:

```
#pwd打印当前目录
exec:
    cd /home/hchen
    pwd
#pwd打印“/home/hchen”
exec:
    cd /home/hchen; pwd
```

4、每当命令运行完后，make会检测每个命令的返回码，如果命令返回成功，那么make会执行下一条命令，当规则中所有的命令成功返回后，这个规则就算是成功完成了。如果一个规则中的某个命令出错了（命令退出码非零），那么make就会终止执行当前规则，这将有可能终止所有规则的执行.

命令出错不代表错误. 在命令前加“-”可以忽略此命令的出错. 一个规则是以“.IGNORE”作为目标的，那么这个规则中的所有命令将会忽略错误. 用“-i”或是“--ignore-errors”参数忽略Makefile中所有出错. 

参数“-k”或是“--keep-going”，这个参数的意思是，如果某规则中的命令出错了，那么就终目该规则的执行，但继续执行其它规则


## 变量

1、变量在Makefile执行的时候会自动原模原样地展开在所使用的地方

2、变量的命名字可以包含字符、数字，下划线（可以是数字开头），但不应该含有“:”、“#”、“=”或是空字符（空格、回车等）。变量是大小写敏感的, 使用大小写搭配的变量名可以避免和系统的变量冲突

3、变量在声明时需要给予初值，而在使用时，需要给在变量名前加上“$”符号，但最好用小括号“（）”或是大括号“{}”把变量给包括起来。如果你要使用真实的“$”字符，那么你需要用“$$”来表示

4、可以使用其它变量来构造变量的值

5、“=”右侧变量的值可以定义在文件的任何一处

6、":="只能使用右侧变量已定义好的值, eg:

```
x := foo
y := $(x) bar
x := later
等价于
y := foo bar
x := later
```

```
y := $(x) bar
x := foo
#y的值是“bar”
```

7、空格变量

```
nullstring :=
space := $(nullstring) # end of the line
```

nullstring是一个Empty变量，其中什么也没有，而我们的space的值是一个空格。因为在操作符的右边是很难描述一个空格的，这里采用的技术很管用，先用一个Empty变量来标明变量的值开始了，而后面采用“#”注释符来表示变量定义的终止

`dir := /foo/bar    # directory to put the frobs in` dir这个变量的值是“/foo/bar”，后面还跟了4个空格，如果我们这样使用这样变量来指定别的目录——“$(dir)/file”那么就完蛋了。

8、`FOO ?= bar` 含义是，如果FOO没有被定义过，那么变量FOO的值就是“bar”，如果FOO先前被定义过，那么这条语将什么也不做

9、我们可以替换变量中的共有的部分，其格式是“$(var:a=b)”或是“${var:a=b}”，其意思是，把变量“var”中所有以“a”字串“结尾”的“a”替换成“b”字串。这里的“结尾”意思是“空格”或是“结束符”, eg:

```
foo := a.o b.o c.o
bar := $(foo:.o=.c)
```

10、把变量的值再当成变量

```
x = y
y = z
a := $($(x))
```

在这个例子中，$(x)的值是“y”，所以$($(x))就是$(y)，于是$(a)的值就是“z”

```
x = variable1
variable2 := Hello
y = $(subst 1,2,$(x))
z = y
a := $($($(z)))
```

subst函数把“variable1”中的所有“1”字串替换成“2”字串, $(a)的值就是$(variable2)的值——“Hello”

---

多个变量来组成一个变量的名字，然后再取其值:

```
first_second = Hello
a = first
b = second
all = $($a_$b)
```

“$a_$b”组成了“first_second”，于是，$(all)的值就是“Hello”

---

```
ifdef do_sort
func := sort
else
func := strip
endif

bar := a d b g q c
foo := $($(func) $(bar))
```

---

```
dir = foo
$(dir)_sources := $(wildcard $(dir)/*.c)
define $(dir)_print
lpr $($(dir)_sources)
endef
```

11、“+=”追加变量值

```
objects = main.o foo.o bar.o utils.o
objects += another.o
```

$(objects)值变成：“main.o foo.o bar.o utils.o another.o”

如果变量之前没有定义过，那么，“+=”会自动变成“=”，如果前面有变量定义，那么“+=”会继承于前次操作的赋值符, :

```
variable := value
variable += more
等价于
variable := value
variable := $(variable) more
```





> 摘录到了[跟我一起写 Makefile（八）](https://blog.csdn.net/haoel/article/details/2893)