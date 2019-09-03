# Stack 0

## （一） 题目：

\#include &lt;stdlib.h&gt;

\#include &lt;unistd.h&gt;

\#include &lt;stdio.h&gt;

\#include &lt;string.h&gt;

void win\(\)

{

printf\("code flow successfully changed\n"\);

}

int main\(int argc, char \*\*argv\)

{

char buffer\[64\];

gets\(buffer\);

}

## （二）解题过程：

gdb调试

64+ 8\(对齐） +4\(ebp\)+e\(ret\)

win 入口地址 0x080483f4

![](/png/11.png)

