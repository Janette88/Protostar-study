# Net2

## （一） 题目：

\#include "../common/common.c"

\#define NAME "net2"

\#define UID 997

\#define GID 997

\#define PORT 2997

void run\(\)

{

unsigned int quad\[4\];

int i;

unsigned int result, wanted;

result = 0;

for\(i = 0; i &lt; 4; i++\) {

```
  quad\[i\] = random\(\);

  result += quad\[i\];



  if\(write\(0, &amp;\(quad\[i\]\), sizeof\(result\)\) != sizeof\(result\)\) {

      errx\(1, ":\(\n"\);

  }
```

}

if\(read\(0, &wanted, sizeof\(result\)\) != sizeof\(result\)\) {

```
  errx\(1, ":&lt;\n"\);
```

}

if\(result == wanted\) {

```
  printf\("you added them correctly\n"\);
```

} else {

```
  printf\("sorry, try again. invalid\n"\);
```

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

/\* Don't do this :&gt; \*/

srandom\(time\(NULL\)\);

run\(\);

}

## （二）解题过程：

本题的关键点看懂源代码

主程序还是启动daemon，等待被客户端连接2997端口，发送4个随机数字到客户端，这四个数字执行相加操作，得到一个总和，然后把总和以str的形式发送回server，如果正确，完成。

[https://youremindmeofmymother.com/2015/09/04/protostar-exploits-net2/](https://youremindmeofmymother.com/2015/09/04/protostar-exploits-net2/)

这个网站解释的清楚，交互。

代码一：（简洁）

from  socket import \*

from struct import \*

s = socket\(AF\_INET,SOCK\_STREAM\)

s.connect\("127.0.0.1",2997\)

sum=0

for i in range\(4\):

```
res = s.recv\(10\)

print res.encode\("hex"\)

sum +=int\(unpack\("&lt;I",res\[0\]\)\)
```

print sum

s.send\(str\(pack\("I",sum\)\)\)

print \(s.recv\(1024\)\)

s.close\(\)

代码2： 详细，有交互的过程

![](/png/88.png)

执行结果如下：

![](/png/89.png)

