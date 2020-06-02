$ cat Makefile

# Makefile
# 目标文件名、依赖文件名，都可以带路径名
# 有a，b，c，这3个目录，
# main.c在a目录，对象.o文件会生成到b目录，可执行文件会生成到c目录。

c/a.out : b/main.o
	cc $^ -o $@

b/main.o : a/main.c
	cc -c $< -o $@

$
$ cat main.c

#include <stdio.h>

int main()
{
    printf("%s:%d\n", __FILE__, __LINE__);
    return 0;
}
$
$ make
$
$ find . -print | sed -e 's;[^/]*/;|____;g;s;____|; |;g'
.
|____Makefile
|____a
| |____main.c
|____c
| |____a.out
|____b
| |____main.o
$ 
