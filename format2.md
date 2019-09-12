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

格式化字符串漏洞是`PWN`题常见的考察点，仅次于栈溢出漏洞。漏洞原因：程序使用了格式化字符串作为参数，并且格式化字符串为用户可控。其中触发格式化字符串漏洞函数主要是`printf`、`sprintf`、`fprintf`、`prin`等C库中`print`家族的函数

### 0×01 格式化字符串介绍

```
printf（"格式化字符串",参数...)
```

该`printf`函数的第一个参数是由格式化说明符与字符串组成，用来规定参数用什么格式输出内容。

格式化说明符：

```
%d - 十进制 - 输出十进制整数
%s - 字符串 - 从内存中读取字符串
%x - 十六进制 - 输出十六进制数
%c - 字符 - 输出字符
%p - 指针 - 指针地址
%n - 到目前为止所写的字符数
```

思路：关键是构造特殊的字串，使得覆盖原来的target地址，覆盖后的值为64即可。

ref:

[https://www.cnblogs.com/ichunqiu/p/9329387.html](https://www.cnblogs.com/ichunqiu/p/9329387.html)

[https://www.freebuf.com/column/207425.html](https://www.freebuf.com/column/207425.html)

[https://blog.51cto.com/terrying/1182328](https://blog.51cto.com/terrying/1182328)

https://www.jianshu.com/p/bced7df926bb

