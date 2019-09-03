# Stack 2

## （一） 题目：

\#include &lt;stdlib.h&gt;

\#include &lt;unistd.h&gt;

\#include &lt;stdio.h&gt;

\#include &lt;string.h&gt;

int main\(int argc, char \*\*argv\)

{

volatile int modified;

char buffer\[64\];

char \*variable;

variable = getenv\("GREENIE"\);

if\(variable == NULL\) {

```
  errx\(1, "please set the GREENIE environment variable\n"\);
```

}

modified = 0;

strcpy\(buffer, variable\);

if\(modified == 0x0d0a0d0a\) {

```
  printf\("you have correctly modified the variable\n"\);
```

} else {

```
  printf\("Try again, you got 0x%08x\n", modified\);
```

}

## （二）解题过程：

函数变量在内存中的分布如上提。栈溢出

variable

buffer\[64\]

modified         0a0d0a0d

argc

argv

用python构造利用代码 exploit.py

注意地址的顺序和字节在内存中的顺序正好相反。

![](/png/10.png)

