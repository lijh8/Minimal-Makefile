$ apt install cmake # ninja-build

$ cat CMakeLists.txt
cmake_minimum_required(VERSION 3.24)
project(myproject VERSION 1.0)
set(CMAKE_C_FLAGS "-std=c99 -g") # -O3 # .cpp
set(CMAKE_BUILD_RPATH_USE_ORIGIN True)
set(CMAKE_INSTALL_RPATH "$ORIGIN/../lib")
#set(CMAKE_BUILD_WITH_INSTALL_RPATH True) # install only
file(GLOB SOURCES *.c) # .cpp

add_executable(main ${SOURCES})
#add_library(main SHARED ${SOURCES})
#set_target_properties(main PROPERTIES VERSION 1.2.3 SOVERSION 1.2)
#target_link_libraries(main PUBLIC foo)
#target_link_directories(main PUBLIC ../foo)
#target_include_directories(main PUBLIC ../foo)
add_subdirectory(./foo)
add_subdirectory(./main)

# make DESTDIR=~/tmp install, remove the dir to uninstall
install(TARGETS foo main)
$

$ cmake -S . -B build
$ cmake --build build

# or,
$ mkdir build; cd build
$ cmake ..
$ make
