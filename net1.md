# Net1

## （一） 题目：

`#include "../common/common.c"`

`#define NAME "net1"`

`#define UID 998`

`#define GID 998`

`#define PORT 2998`

`void run()`

`{`

`char buf[12];`

`char fub[12];`

`char *q;`

`unsigned int wanted;`

`wanted = random();`

`sprintf(fub, "%d", wanted);`

`if(write(0,`

`&`

`amp;wanted, sizeof(wanted)) != sizeof(wanted)) {`

`errx(1, ":(\n");`

`}`

`if(fgets(buf, sizeof(buf)-1, stdin) == NULL) {`

`errx(1, ":(\n");`

`}`

`q = strchr(buf, '\r'); if(q) *q = 0;`

`q = strchr(buf, '\n'); if(q) *q = 0;`

`if(strcmp(fub, buf) == 0) {`

`printf("you correctly sent the data\n");`

`} else {`

`printf("you didn't send the data properly\n");`

`}`

`}`

`int main(int argc, char **argv, char **envp)`

`{`

`int fd;`

`char *username;`

`/* Run the process as a daemon */`

`background_process(NAME, UID, GID);`

`/* Wait for socket activity and return */`

`fd = serve_forever(PORT);`

`/* Set the client socket to STDIN, STDOUT, and STDERR */`

`set_io(fd);`

`/* Don't do this :`

`>`

`*/`

`srandom(time(NULL));`

`run();`

`}`

## （二）解题过程：

跟上题类似，主要是1） 把服务器端发的数据以ascii码接收到 2）转换成16进制的形式 3）以小端的顺序读出来 4）再转换成10进制

所以关键是熟练掌握python网络编程

![](/png/86.png)

脚本运行如下：

![](/png/87.png)

