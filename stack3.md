# Stack 3

## （一） 题目：

\#include &lt;stdlib.h&gt;

\#include &lt;unistd.h&gt;

\#include &lt;stdio.h&gt;

\#include &lt;string.h&gt;

void win\(\)

{

  printf\("code flow successfully changed\n"\);

}

int main\(int argc, char \*\*argv\)

{

  volatile int \(\*fp\)\(\);

  char buffer\[64\];

  fp = 0;

  gets\(buffer\);

  if\(fp\) {

      printf\("calling function pointer, jumping to 0x%08x\n", fp\);

      fp\(\);

  }

}

## （二）解题过程：



