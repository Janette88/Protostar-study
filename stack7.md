# Stack 7

## （一） 题目：

```
   \#include &lt;stdlib.h&gt;
```

\#include &lt;unistd.h&gt;

\#include &lt;stdio.h&gt;

\#include &lt;string.h&gt;

char \*getpath\(\)

{

char buffer\[64\];

unsigned int ret;

printf\("input path please: "\); fflush\(stdout\);

gets\(buffer\);

ret = \_\_builtin\_return\_address\(0\);

if\(\(ret & 0xb0000000\) == 0xb0000000\) {

```
  printf\("bzzzt \(%p\)\n", ret\);

  \_exit\(1\);
```

}

printf\("got path %s\n", buffer\);

return strdup\(buffer\);

}

int main\(int argc, char \*\*argv\)

{

getpath\(\);

}

## （二）解题过程：

如图：![](/png/41.png)

思路：

1） 参照stack5 stack6的方法处理 栈溢出

2）调试coredump的方法

 不同于stack6，本题限制返回地址不可以是库函数，比如不可是system\(\).这个限制仅仅限制在第一次返回时候，所以意味着，可以利用ROP方法，用一个gadget,后面可以正常跟shellcode



