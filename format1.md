# format1

## （一） 题目：

  源代码：

  \#include &lt;stdlib.h&gt;

\#include &lt;unistd.h&gt;

\#include &lt;stdio.h&gt;

\#include &lt;string.h&gt;

int target;

void vuln\(char \*string\)

{

printf\(string\);



if\(target\) {

    printf\("you have modified the target :\)\n"\);

}

}

int main\(int argc, char \*\*argv\)

{

vuln\(argv\[1\]\);

}

## （二）解题过程：

ref:

[http://www.atomsec.org/iot/tew654-命令执行漏洞/](http://www.atomsec.org/iot/tew654-命令执行漏洞/)

[http://sh3llc0d3r.com/protostar-exploit-exercises-format1/](http://sh3llc0d3r.com/protostar-exploit-exercises-format1/)

