# format2

## （一） 题目：

\#include &lt;unistd.h&gt;

\#include &lt;stdio.h&gt;

\#include &lt;string.h&gt;

int target;

void vuln\(\)

{

char buffer\[512\];

fgets\(buffer, sizeof\(buffer\), stdin\);

printf\(buffer\);

if\(target == 64\) {

```
  printf\("you have modified the target :\)\n"\);
```

} else {

```
  printf\("target is %d :\(\n", target\);
```

}

}

int main\(int argc, char \*\*argv\)

{

vuln\(\);

}

## （二）解题过程：

这道题需要了解格式化字符串漏洞的原理以及运用。

ref:

[https://www.cnblogs.com/ichunqiu/p/9329387.html](https://www.cnblogs.com/ichunqiu/p/9329387.html)

https://www.freebuf.com/column/207425.html

