$ apt install cmake

$ vi CMakeLists.txt
cmake_minimum_required(VERSION 3.18)
project(MyProject VERSION 1.0)
set(CMAKE_BUILD_RPATH_USE_ORIGIN TRUE)
set(CMAKE_INSTALL_RPATH $ORIGIN/../lib)
#set(CMAKE_C_COMPILER gcc) # .cpp
set(CMAKE_C_STANDARD 11) # .cpp
set(CMAKE_C_STANDARD_REQUIRED TRUE) # .cpp
if (CMAKE_BUILD_TYPE STREQUAL Debug)
  set(CMAKE_C_FLAGS ${CMAKE_C_FLAGS} -fsanitize=address) # .cpp
endif()
file(GLOB SOURCES *.c) # .cpp

add_executable(main ${SOURCES})
#add_library(main SHARED ${SOURCES})
#set_target_properties(main PROPERTIES VERSION 1.2.3 SOVERSION 1.2)
target_link_libraries(server PUBLIC foo)
target_link_directories(server PUBLIC ../foo)
target_include_directories(server PUBLIC ../foo)
#add_subdirectory(./foo)

# make -C build DESTDIR=~/MyProject install
install(TARGETS main # foo
    )
#file(GLOB DYN_LIB ../lib/*so*)
#install(FILES ${DYN_LIB} DESTINATION lib
#    )
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

$ cmake -DCMAKE_BUILD_TYPE=Debug -S . -B build # Release
$ cmake --build build -v # verbose
$ make -C build DESTDIR=~/myproject install

## or,
$ mkdir build; cd build;
$ cmake -DCMAKE_BUILD_TYPE=Debug .. # Release
$ make -s # silent
$ make DESTDIR=~/myproject install
