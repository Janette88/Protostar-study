# Stack 7

## （一） 题目：

       \#include &lt;stdlib.h&gt;

\#include &lt;unistd.h&gt;

\#include &lt;stdio.h&gt;

\#include &lt;string.h&gt;



char \*getpath\(\)

{

  char buffer\[64\];

  unsigned int ret;

  printf\("input path please: "\); fflush\(stdout\);

  gets\(buffer\);

  ret = \_\_builtin\_return\_address\(0\);



  if\(\(ret & 0xb0000000\) == 0xb0000000\) {

      printf\("bzzzt \(%p\)\n", ret\);

      \_exit\(1\);

  }

  printf\("got path %s\n", buffer\);

  return strdup\(buffer\);

}

int main\(int argc, char \*\*argv\)

{

  getpath\(\);

}

## （二）解题过程：



