https://cmake.org/cmake/help/book/mastering-cmake/
https://cmake.org/cmake/help/latest/

https://cmake.org/features/
Target-centric approach

Modern CMake allows the build to be specified as a set of targets (executables,
libraries, custom commands). Each target defines what other targets it depends
on directly. CMake then performs the build in the correct order and includes
transitive link targets.

$ pwd
/home/user1/doc/myapp/src
$ ls
CMakeLists.txt  foo  main
$ tree
.
├── CMakeLists.txt
├── foo
│   ├── CMakeLists.txt
│   ├── foo.c
│   └── foo.h
└── main
    ├── CMakeLists.txt
    └── main.c

$

$ cat CMakeLists.txt
cmake_minimum_required(VERSION 3.31)
project(myapp VERSION 1.0)
set(CMAKE_C_COMPILER gcc) # .cpp
set(CMAKE_C_STANDARD 11) # .cpp
set(CMAKE_BUILD_RPATH_USE_ORIGIN TRUE) # boolean set(): TRUE/FALSE
set(CMAKE_INSTALL_RPATH "$ORIGIN/../lib")

# $ cmake -B build -S . -DENABLE_SANITIZERS=ON
option(ENABLE_SANITIZERS "Enable Sanitizers" OFF) # option(): ON/OFF
if (ENABLE_SANITIZERS)
    add_compile_options(-g)
    add_compile_options(-fsanitize=address,undefined)
    add_link_options(-fsanitize=address,undefined)
endif()

add_subdirectory(main)
add_subdirectory(foo)

# $ make -C build DESTDIR=~/myapp install
install(TARGETS main)
install(TARGETS foo)
#file(GLOB LIBRARIES ../lib/*.so*) # external libraries
#install(FILES ${LIBRARIES} DESTINATION lib)
$

$ cat foo/CMakeLists.txt
file(GLOB SOURCES *.c) # .cpp
add_library(foo SHARED ${SOURCES})
target_include_directories(foo PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
set_target_properties(foo PROPERTIES VERSION 1.2.3 SOVERSION 1)
$

$ cat main/CMakeLists.txt
file(GLOB SOURCES *.c) # .cpp
add_executable(main ${SOURCES})
target_link_libraries(main PRIVATE foo)
$

$ cat foo/foo.c
#include <limits.h>
#include <stdbool.h>
#include "foo.h"

bool add(int a, int b, int *result)
{
    // if (b < 0 && a < INT_MIN - b || b > 0 && a > INT_MAX - b)
    //     return false;

    *result = a + b;
    return true;
}
$
$ cat foo/foo.h
#ifndef foo_h
#define foo_h

#include <stdbool.h>

bool add(int a, int b, int *result);

#endif
$
$ cat main/main.c
#include <stdio.h>
#include <limits.h>
#include <stdbool.h>
#include "foo.h"

int main()
{
    int sum;
    int a = INT_MAX;
    int b = INT_MAX;
    bool ret = add(a, b, &sum);

    // /home/ljh/doc/myapp/src/foo/foo.c:10:17: runtime error: signed integer overflow: 2147483647 + 2147483647 cannot be represented in type 'int'
}
$

$ apt install cmake

$ cmake -B build -S . # -DENABLE_SANITIZERS=ON
$ cmake -B build -S . # -DCMAKE_BUILD_TYPE=Debug # Release
$ cmake --build build -j4 # --target clean # -v # verbose
$ make -C build DESTDIR=~/myapp install # staged install

## or,
$ mkdir build && cd build
$ cmake .. # -DCMAKE_BUILD_TYPE=Debug # Release
$ make -j4 # -s # silent
$ make DESTDIR=~/myapp install


---


# another example

$ vi CMakeLists.txt

cmake_minimum_required(VERSION 3.18)
project(myapp VERSION 1.0)
set(CMAKE_BUILD_RPATH_USE_ORIGIN TRUE)
set(CMAKE_INSTALL_RPATH $ORIGIN/../lib)

if (CMAKE_BUILD_TYPE STREQUAL Debug)
    add_link_options(-fsanitize=address)
endif()
#link_libraries(-lfoo) # linker flag -l
#link_directories(../foo)
#include_directories(../foo)
#add_compile_definitions(VAR) # VAR=value
add_compile_options(-std=c11) # .cpp
#set(CMAKE_C_COMPILER gcc) # .cpp
file(GLOB SOURCES *.c) # .cpp

add_executable(main ${SOURCES})
#add_library(foo SHARED ${SOURCES})
#set_target_properties(foo PROPERTIES VERSION 1.2.3 SOVERSION 1)
#add_subdirectory(./foo)

# make -C build DESTDIR=~/foo install
install(TARGETS main) # foo
#file(GLOB LIBRARIES ../lib/*.so*)
#install(FILES ${LIBRARIES} DESTINATION lib)

$

Note:

link_libraries(-lfoo) # foo is a file, eg.: libfoo.so, or, libfoo.a,

link_libraries(foo) # foo is a cmake target.


---


// LSan normally runs only at program exit.
// In a forever loop long-running service process,
// the program never exits, so memory leaks are never reported.
// Therefore, manual triggers (SIGCONT + __lsan_do_recoverable_leak_check()) are
// the only practical way to detect leaks during operation.

// $ nohup ./a.out &
// $ kill -CONT 12345 # pid

$ vi main.c
#include <sanitizer/lsan_interface.h>

void handlerCont(int signum) {
    printf("SIGCONT %d\n", signum);
#ifndef NDEBUG
    __lsan_do_recoverable_leak_check();
#endif
}

int main() {
    signal(SIGCONT, handlerCont); // $ man 7 signal
    //...
}


---


PUBLIC: 当前目标用，也传播给链接到当前目标的使用者
PRIVATE: 当前目标自己用，不传播给链接到当前目标的使用者
INTERFACE: 当前目标自己不用，传播给链接到当前目标的使用者

foo/CMakeLists.txt
target_link_libraries(foo PUBLIC bar)                # A - 建立依赖关系
target_include_directories(foo PUBLIC /foo/include)  # B - 设置包含目录
target_compile_definitions(foo PUBLIC -DFOO_ENABLED) # C - 设置编译定义

main/CMakeLists.txt
target_link_libraries(main PUBLIC foo)               # 这行触发传播
                                                     # main -> foo -> bar
                                                     # bar通过foo, 被main访问;
                                                     # main是foo的使用者.


---


CMake target-level link:

target_link_libraries(main PUBLIC foo)

. 这是 Modern CMake 的目标级别链接;

. 使用被链接目标中的完整库文件路径 ../foo/libfoo.so.1.2.3 ,
  而非使用 ` -L/path/to `, ` -lfoo ` ,
  这适用于静态库 ` .a `, 或动态共享库 ` .so ` ;

. Propagate transitive usage requirements,
  https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#target-usage-requirements ,
  传递被链接目标中 target_include_directories 命令,
  其中的 PUBLIC, INTERFACE 公开的头文件包含路径,
  自动获得头文件包含路径 include ` -I/header/include/path/ ` ;

. 管理依赖，先构建被链接目标;


---


执行配置:
当修改/增/删 CMakeLists.txt, 增/删 C, C++ 文件或目录结构时;
$ cmake -B build -S . # -DCMAKE_BUILD_TYPE=Debug # Release

执行构建:
当只修改了现有 C, C++ 文件中代码时;
$ cmake --build build -j4 # --target clean # -v # verbose


---
