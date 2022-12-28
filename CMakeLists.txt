$ apt install cmake # ninja-build

$ cat CMakeLists.txt
cmake_minimum_required(VERSION 3.24)
project(myproject VERSION 1.0)
set(CMAKE_BUILD_RPATH_USE_ORIGIN True)
set(CMAKE_INSTALL_RPATH "$ORIGIN/../lib")
set(CMAKE_C_STANDARD 11) # .cpp
set(CMAKE_C_STANDARD_REQUIRED True) # .cpp
file(GLOB SOURCES *.c) # .cpp

add_executable(main ${SOURCES})
#add_library(main SHARED ${SOURCES})
#set_target_properties(main PROPERTIES VERSION 1.2.3 SOVERSION 1.2)
#target_link_libraries(main PUBLIC foo)
#target_link_directories(main PUBLIC ../foo)
#target_include_directories(main PUBLIC ../foo)
#add_subdirectory(./foo)

# remove the DESTDIR to uninstall
install(TARGETS main # foo
    )
$

# build type: Debug, Release
$ cmake -DCMAKE_BUILD_TYPE=Debug -S . -B build
$ cmake --build build -v # verbose

# or,
$ mkdir build; cd build; cmake -DCMAKE_BUILD_TYPE=Debug ..
$ make -s # silent

$ make DESTDIR=~/tmp -C build install
