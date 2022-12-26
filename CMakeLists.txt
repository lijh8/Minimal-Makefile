$ apt install cmake # ninja-build

$ cat CMakeLists.txt
cmake_minimum_required(VERSION 3.24)
project(myproject VERSION 1.0)
set(CMAKE_BUILD_RPATH_USE_ORIGIN True)
set(CMAKE_C_FLAGS "-std=c11 -g") # -O3 # .cpp
#file(GLOB SOURCES *.c) # .cpp

add_executable(main ${SOURCES})
#add_library(main SHARED ${SOURCES})
#target_link_libraries(main PUBLIC foo)
#target_link_directories(main PUBLIC ../foo)
#target_include_directories(main PUBLIC ../foo)
#add_subdirectory(./foo)
$

$ cat cmake_clean
find . \( -name CMakeFiles \
       -o -name cmake_install.cmake \
       -o -name Makefile \
       -o -name CMakeCache.txt \) -exec rm -fr {} \;
$

$ cmake -S . -B build
$ cmake --build build

# or,
$ mkdir build && cd build
$ cmake ..
$ make
