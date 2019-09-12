# format2

## （一） 题目：

  \#include &lt;unistd.h&gt;

\#include &lt;stdio.h&gt;

\#include &lt;string.h&gt; 

int target;

void vuln\(\)

{

  char buffer\[512\];

  fgets\(buffer, sizeof\(buffer\), stdin\);

  printf\(buffer\);

  if\(target == 64\) {

      printf\("you have modified the target :\)\n"\);

  } else {

      printf\("target is %d :\(\n", target\);

  }

}

int main\(int argc, char \*\*argv\)

{

  vuln\(\);

}



