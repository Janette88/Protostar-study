# format0

## （一） 题目：

\#include &lt;stdlib.h&gt;

\#include &lt;unistd.h&gt;

\#include &lt;stdio.h&gt;

\#include &lt;string.h&gt;

void vuln\(char \*string\)

{

volatile int target;

char buffer\[64\];

target = 0;

sprintf\(buffer, string\);

if\(target == 0xdeadbeef\) {

```
  printf\("you have hit the target correctly :\)\n"\);
```

}

}

int main\(int argc, char \*\*argv\)

{

vuln\(argv\[1\]\);

}

## （二）解题过程：


