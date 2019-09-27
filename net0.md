# Net0

## （一） 题目：

\#include "../common/common.c"

\#define NAME "net0"

\#define UID 999

\#define GID 999

\#define PORT 2999

void run\(\)

{

unsigned int i;

unsigned int wanted;

wanted = random\(\);

printf\("Please send '%d' as a little endian 32bit int\n", wanted\);

if\(fread\(&i, sizeof\(i\), 1, stdin\) == NULL\) {

```
  errx\(1, ":\(\n"\);
```

}

if\(i == wanted\) {

```
  printf\("Thank you sir/madam\n"\);
```

} else {

```
  printf\("I'm sorry, you sent %d instead\n", i\);
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

本题主要是考验读代码和写代码的能力.题目给出一个随机生成的数字，需要与另一个程序交互，当对方输入的数字跟它一致时候，就可以建立连接。

源代码是一个网络daemon程序，监听2999.这个daemon程序自动启动。

![](/assets/82.png)

netstat -an\| grep net0

执行nc -vvv localhost 2999

![](/assets/83.png)

要解决的问题就是编写一个程序和源代码进行交互，实现将程序读出来的数字 转换成16进制低端顺序，然后发送回服务端。

主要代码：

1）创建一个跟服务端的链接

`s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)`

`remote_ip = socket.gethostbyname( host )`

`s.connect((remote_ip, int(port)))`

2）从服务端接收数据

`recv = s.recv(4096)`

在接收的数据中查找10进制的数字

3）将找到的10进制字串转换成16进制，并去掉0x字符

`hexval = hex(int(found))`

`hexval = hexval[2:]#remove leading '0x' characters`

4）从大端转换成小端

`rev_bytes = struct.pack('<L',int(found))#Convert big endian to little`

5）转换好的数据发送到服务端

`s.sendall(rev_bytes)`

服务端接受数据

实现代码如下：

![](/assets/84.png)

执行结果如下：

![](/png/85.png)

ref:

[https://youremindmeofmymother.com/2015/09/04/protostar-exploits-net0/](https://youremindmeofmymother.com/2015/09/04/protostar-exploits-net0/)

[http://www.voidcn.com/article/p-vrmclykc-xo.html](http://www.voidcn.com/article/p-vrmclykc-xo.html)

