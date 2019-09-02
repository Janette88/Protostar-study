# Stack 0

## （一） 题目：

This level introduces the concept that memory can be accessed outside of its allocated region, how the stack variables are laid    out, and that modifying outside of the allocated memory can modify program execution.

This level is at /opt/protostar/bin/stack0

source code:

\#include &lt;stdlib.h&gt;

\#include &lt;unistd.h&gt;

\#include &lt;stdio.h&gt;

int main\(int argc, char \*\*argv\)

{

volatile int modified;

char buffer\[64\];

modified = 0;

gets\(buffer\);

if\(modified != 0\) {

```
  printf\("you have changed the 'modified' variable\n"\);
```

} else {

```
  printf\("Try again?\n"\);
```

}

}

## （二）解题过程：

```
 This problem is about how to layout the stack in memory
```

Ref：

[https://exploit.education/phoenix/stack-zero/](https://exploit.education/phoenix/stack-zero/)

https://blog.csdn.net/guilanl/article/details/61916375

