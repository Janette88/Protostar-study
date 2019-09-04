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

