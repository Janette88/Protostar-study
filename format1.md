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

格式化字符串漏洞其实是一种很基本的漏洞类型,成因是printf\(\)函数的参数只给定了需要打印的str,比如这样

printf\(str\);

那么格式化字符串本身,也就是我们的%d,%s,%x,就变成用户可控的情况了。

这里要注意的就是,printf其实算是一个接受变参的函数,格式化字符串之后的

比如

printf\("%x%x",a,b\);

但是这个时候,我们把b去掉 只保留a,却发现仍然打印出了一个数值。

很明显,printf的原理是直接用esp做基准去取参数,所以说这也是比较明显的一个问题了,我们完全可以任意读取其他地址的内存。

而且%n更是提供了写入功能,以参数对应的数值为指针,写入已经被打印出的字符数量。一个比较完整的介绍是。

%\[第几个参数\]$\[对齐格式,比如010是用0补齐,10长度\]x

因此,我们可以通过向第n个参数写入一个地址的方法,再把x换成n,就能写入了。

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

2）确定存放target地址的地方和此时的栈顶esp距离

gdb调试，在vuln下断点，代入参数 python的方法

![](/png/54.png)

如上图，此时target数值的地方在0xbffffe89处

0xbffffe89-0xbffffc70= 537

537/4=134\(word\)+1

从esp地方填充137个字+1 ,就到了target地方。

python -c 'print "\x37\x96\x04\x08A%134$n"'

注意在调试过程中会出现地址变化的情况，执行一次成功后，如果还想再调试，最好重启机器。

./format1 \`python -c 'print "\x38\x96\x04\x08AAA%134$x"'\`

下图是重启以后的结果

![](/png/55.png)

ref:

[http://www.atomsec.org/iot/tew654-命令执行漏洞/](http://www.atomsec.org/iot/tew654-命令执行漏洞/)

[http://sh3llc0d3r.com/protostar-exploit-exercises-format1/](http://sh3llc0d3r.com/protostar-exploit-exercises-format1/)

[https://zhuanlan.zhihu.com/p/23197261](https://zhuanlan.zhihu.com/p/23197261)

http://www.pwntester.com/blog/2013/12/19/protostar-format0-4-write-ups/

