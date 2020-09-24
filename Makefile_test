$ cat Makefile
# Makefile
# target files or dependency files can all take paths with them
# eg. 3 folders: a, b ,c
#   main.c   in a,
#   main.o   in b,
#   main.out in c

c/main.out : b/main.o
	mkdir -p $(dir $@)
	cc $^ -o $@

b/main.o : a/main.c
	mkdir -p $(dir $@)
	cc -c $< -o $@

$ cat a/main.c
#include <stdio.h>

int main()
{
    printf("hello\n");
    return 0;
}

$ make

$ tree .
.
├── a
│   └── main.c
├── b
│   └── main.o
├── c
│   └── main.out
└── Makefile

$
