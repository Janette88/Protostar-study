# Stack 1

## （一） 题目：

\#include &lt;stdlib.h&gt;

\#include &lt;unistd.h&gt;

\#include &lt;stdio.h&gt;

\#include &lt;string.h&gt;

int main\(int argc, char \*\*argv\)

{

volatile int modified;

char buffer\[64\];

if\(argc == 1\) {

```
  errx\(1, "please specify an argument\n"\);
```

}

modified = 0;

strcpy\(buffer, argv\[1\]\);

if\(modified == 0x61626364\) {

```
  printf\("you have correctly got the variable to the right value\n"\);
```

} else {

```
  printf\("Try again, you got 0x%08x\n", modified\);
```

}

}

## （二）解题过程：

   这也是一道缓冲区溢出的题。根据之前的原理：



     



