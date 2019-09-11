# format1

## （一） 题目：

源代码：

\#include &lt;stdlib.h&gt;

\#include &lt;unistd.h&gt;

\#include &lt;stdio.h&gt;

\#include &lt;string.h&gt;

int target;

void vuln\(char \*string\)

{

printf\(string\);

if\(target\) {

```
printf\("you have modified the target :\)\n"\);
```

}

}

int main\(int argc, char \*\*argv\)

{

vuln\(argv\[1\]\);

}

## （二）解题过程：

本题的思路也比较简单，通过读取源码，主函数调用vuln函数，vuln函数通过传入的参数控制，传的参数就是字符串，通过传入的字符串覆盖target全局变量位置。

关键点：

1） 获取target的位置

2）获取当前栈顶到target的距离 多少字节 （或者多少字）

3） 格式字符串

printf\("%s",string\) 和 printf\(string\)是非常不一样的，后者存在漏洞，在构造string时候可以控制，比如后面跟着%x时候就可以输出堆栈的内容。

思路验证：

1\) 定位target的地址有两种方法

用objdump -t ./format1 \| grep target 

这样可以找到程序 .bss段里 target的地址

![](/png/52.png)

另外一种方法用gdb找target地址

![](/png/53.png)



ref:

[http://www.atomsec.org/iot/tew654-命令执行漏洞/](http://www.atomsec.org/iot/tew654-命令执行漏洞/)

[http://sh3llc0d3r.com/protostar-exploit-exercises-format1/](http://sh3llc0d3r.com/protostar-exploit-exercises-format1/)

