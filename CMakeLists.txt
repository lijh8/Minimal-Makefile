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
