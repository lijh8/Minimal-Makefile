$ apt install cmake

$ vi CMakeLists.txt

cmake_minimum_required(VERSION 3.18)
project(MyProject VERSION 1.0)
set(CMAKE_BUILD_RPATH_USE_ORIGIN TRUE)
set(CMAKE_INSTALL_RPATH $ORIGIN/../lib)

if (CMAKE_BUILD_TYPE STREQUAL Debug)
  add_link_options(-fsanitize=address)
endif()
#link_libraries(foo)
#link_directories(../foo)
#include_directories(../foo)
#add_compile_definitions(VAR) # VAR=value
add_compile_options(-std=c11) # .cpp
#set(CMAKE_C_COMPILER gcc) # .cpp
file(GLOB SOURCES *.c) # .cpp

add_executable(main ${SOURCES})
#add_library(foo SHARED ${SOURCES})
#set_target_properties(foo PROPERTIES VERSION 1.2.3 SOVERSION 1.2)
#add_subdirectory(./foo)

# make -C build DESTDIR=~/foo install
install(TARGETS main) # foo
#file(GLOB LIBRARIES ../lib/*.so*)
#install(FILES ${LIBRARIES} DESTINATION lib)

$

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

$ cmake -S . -B build -DCMAKE_BUILD_TYPE=Debug # Release
$ cmake --build build -v # verbose
$ make -C build DESTDIR=~/MyProject install

## or,
$ mkdir build; cd build;
$ cmake .. -DCMAKE_BUILD_TYPE=Debug # Release
$ make -s # silent
$ make DESTDIR=~/MyProject install


---


$ pwd
/home/ljh/doc/myapp/src
$
$ ls
CMakeLists.txt  build  foo  main
$
$ cat CMakeLists.txt
cmake_minimum_required(VERSION 3.31)
project(myapp VERSION 1.0)
add_subdirectory(foo)
add_subdirectory(main)
$
$ ls foo/
CMakeLists.txt  foo.cpp  foo.h
$
$ cat foo/CMakeLists.txt
file(GLOB SOURCES *.cpp)
add_library(foo SHARED ${SOURCES}) # 不推荐使用GLOB变量，推荐明确列出源文件
target_include_directories(foo PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
$
$ ls main/
CMakeLists.txt  main.cpp
$
$ cat main/CMakeLists.txt
file(GLOB SOURCES *.cpp)
add_executable(main ${SOURCES}) # 不推荐使用GLOB变量，推荐明确列出源文件
target_link_libraries(main PRIVATE foo)
$

$ cat foo/foo.cpp
#include <iostream>
#include "foo.h"

void foo()
{
    std::cout << __func__ << "\n";
}
$
$ cat foo/foo.h
#ifndef foo_h
#define foo_h

void foo();

#endif
$
$ cat main/main.cpp
#include <iostream>
#include "foo.h"

int main()
{
    std::cout << __func__ << "\n";
    foo();
}
$


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


---


target_link_libraries(main PUBLIC foo)远多于GNU Makefile的 -l 和 -L ：

CMake自动处理：
1. 链接库文件（相当于 -lfoo）
2. 添加库的包含路径（相当于 -I/path/to/foo）
3. 添加库的链接路径（相当于 -L/path/to/foo）
4. 传递所有PUBLIC/INTERFACE属性
5. 处理依赖库的依赖
6. 平台特定的链接标志
7. 构建顺序依赖


---


运行配置:
当修改/增/删 CMakeLists.txt, 增/删 C, C++ 文件或目录结构时;

运行构建:
当只修改了现有 C, C++ 文件中代码时;

$ cmake -B build -S .
$ cmake --build build -v -j4

$ mkdir build && cd build
$ cmake ..
$ make -j4


---
