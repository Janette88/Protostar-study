# Stack 6

## （一） 题目：

\#include &lt;stdlib.h&gt;

\#include &lt;unistd.h&gt;

\#include &lt;stdio.h&gt;

\#include &lt;string.h&gt;

void getpath\(\)

{

char buffer\[64\];

unsigned int ret;

printf\("input path please: "\); fflush\(stdout\);

gets\(buffer\);

ret = \_\_builtin\_return\_address\(0\);

if\(\(ret & 0xbf000000\) == 0xbf000000\) {

```
printf\("bzzzt \(%p\)\n", ret\);

\_exit\(1\);
```

}

printf\("got path %s\n", buffer\);

}

int main\(int argc, char \*\*argv\)

{

getpath\(\);

}

## （二）解题过程：

这个是典型的的ROP ，绕过栈保护的一种方法。ret2libc意思是返回到c库。调用system函数执行/bin/sh 然后用exit\(\)退出。最终攻击的payload会是：

填充padding --&gt; system（）地址--&gt;exit\(\)地址----&gt;/bin/sh

1\)判断stack6是否有suid权限

![](/png/26.png)

ref：

[https://blog.csdn.net/stonesharp/article/details/38402953?utm\_source=blogxgwz4](https://blog.csdn.net/stonesharp/article/details/38402953?utm_source=blogxgwz4)

[https://programlife.net/](https://programlife.net/)

[https://0xrick.github.io/binary-exploitation/bof6/](https://0xrick.github.io/binary-exploitation/bof6/)

