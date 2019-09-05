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

stack6 有suid权限。

2）首先，调用system后，需要执行/bin/sh.要实现这个，需要两步：

把/bin/sh放到环境变量里 $shell

然后system（）调用该变量。

方法：

export SHELL=/bin/sh

echo $SHELL

要想得到变量$SHELL的地址，需要用到gdb调试器

b main  在main函数下断点。

r   运行

查看环境变量的地址和值

x/s \* \(\(char \*\*\)environ+x\)   x是一个数字，然后就打印出相应的变量地址

![](/png/27.png)

3）知道了变量的地址后，来获取漏洞溢出的偏移位置。

![](/png/28.png)

4）gdb 执行进入输入路径填充

![](/png/29.png)







ref：

[https://blog.csdn.net/stonesharp/article/details/38402953?utm\_source=blogxgwz4](https://blog.csdn.net/stonesharp/article/details/38402953?utm_source=blogxgwz4)

[https://programlife.net/](https://programlife.net/)

[https://0xrick.github.io/binary-exploitation/bof6/](https://0xrick.github.io/binary-exploitation/bof6/)

[https://www.shellblade.net/posts.html](https://www.shellblade.net/posts.html)

