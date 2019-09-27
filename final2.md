# final2

## （一） 题目：

\#include "../common/common.c"

\#include "../common/malloc.c"

\#define NAME "final2"

\#define UID 0

\#define GID 0

\#define PORT 2993

\#define REQSZ 128

void check\_path\(char \*buf\)

{

char \*start;

char \*p;

int l;

/\*

\* Work out old software bug

\*/

p = rindex\(buf, '/'\);

l = strlen\(p\);

if\(p\) {

start = strstr\(buf, "ROOT"\);

if\(start\) {

while\(\*start != '/'\) start--;

memmove\(start, p, l\);

printf\("moving from %p to %p \(exploit: %s / %d\)\n", p, start, start &lt; buf ?

"yes" : "no", start - buf\);

}

}

}

int get\_requests\(int fd\)

{

char \*buf;

char \*destroylist\[256\];

int dll;

int i;

dll = 0;

while\(1\) {

if\(dll &gt;= 255\) break;

buf = calloc\(REQSZ, 1\);

if\(read\(fd, buf, REQSZ\) != REQSZ\) break;

if\(strncmp\(buf, "FSRD", 4\) != 0\) break;

check\_path\(buf + 4\);

dll++;

}

for\(i = 0; i &lt; dll; i++\) {

write\(fd, "Process OK\n", strlen\("Process OK\n"\)\);

free\(destroylist\[i\]\);

}

}

int main\(int argc, char \*\*argv, char \*\*envp\)

{

int fd;

char \*username;

/\* Run the process as a daemon \*/

background\_process\(NAME, UID, GID\);

/\* Wait for socket activity and return \*/

fd = serve\_forever\(PORT\);

/\* Set the client socket to STDIN, STDOUT, and STDERR \*/

set\_io\(fd\);

get\_requests\(fd\);

}

## （二）解题过程：

```
本题是一道典型的heap unlink漏洞。堆内存可通过malloc()类函数分配，由free()类函数释放，程序的堆内存是由堆管理器分配的。由malloc()分配的内存块是有边界标识（Boundary tag）的。这些位置包含的信息是内存中这个块之前和之后放被放置的两个数据块的大小
```

实际上执行的操作就是：

\*\(P-&gt;fd+12\)=P-&gt;bk;

\*\(P-&gt;bk+8\)=P-&gt;fd

数据块后部指针中的地址可以被写到前部指针加12所指向的位置。如果\*\*\*者可以覆盖这两个指针并调用unlink\(\)的话，那么就能用任何东西覆盖内存的任何位置！！！

本题主要在于exploit脚本的构造。

ref:

https://blog.csdn.net/weixin\_34161032/article/details/92283796

