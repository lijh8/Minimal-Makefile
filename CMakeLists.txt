https://cmake.org/cmake/help/latest/guide/tutorial/
https://cliutils.gitlab.io/modern-cmake/chapters/intro/running.html

$ apt install cmake # ninja-build

# main1 -> libfoo.so -> libbar.so
# main2 -> libfoo.so -> libbar.so

$ ls src
bar  CMakeLists.txt  foo  main1  main2
$

$ find . | sed -e 's;[^/]*/;|____;g;s;____|; |;g'
.
|____.vscode
| |____launch.json
| |____settings.json
| |____tasks.json
|____src
| |____foo
| | |____CMakeLists.txt
| | |____foo.c
| | |____foo.h
| |____main1
| | |____main1.c
| | |____CMakeLists.txt
| |____bar
| | |____bar.c
| | |____bar.h
| | |____CMakeLists.txt
| |____CMakeLists.txt
| |____main2
| | |____main2.c
| | |____CMakeLists.txt
$

#---

$ cat src/CMakeLists.txt
cmake_minimum_required(VERSION 3.24)
project(myproject VERSION 1.0)
set(CMAKE_C_FLAGS "-std=c11 -g") # -O3 # .cpp
#file(GLOB SOURCES *.c) # .cpp

#add_executable(main ${SOURCES})
#add_library(hello SHARED ${SOURCES})
#target_link_libraries(main PUBLIC hello)
#target_link_directories(main PUBLIC ./hello)
#target_include_directories(main PUBLIC ./hello)
add_subdirectory(bar)
add_subdirectory(foo)
add_subdirectory(main1)
add_subdirectory(main2)
$

$ cat src/bar/CMakeLists.txt
#cmake_minimum_required(VERSION 3.24)
#project(myproject VERSION 1.0)
#set(CMAKE_C_FLAGS "-std=c11 -g") # -O3 # .cpp
file(GLOB SOURCES *.c) # .cpp

#add_executable(bar ${SOURCES})
add_library(bar SHARED ${SOURCES})
#target_link_libraries(bar PUBLIC xxx)
#target_link_directories(bar PUBLIC ../xxx)
#target_include_directories(bar PUBLIC ../xxx)
#add_subdirectory(./xxx)
$

$ cat src/foo/CMakeLists.txt
#cmake_minimum_required(VERSION 3.24)
#project(myproject VERSION 1.0)
#set(CMAKE_C_FLAGS "-std=c11 -g") # -O3 # .cpp
file(GLOB SOURCES *.c) # .cpp

#add_executable(foo ${SOURCES})
add_library(foo SHARED ${SOURCES})
target_link_libraries(foo PUBLIC bar)
target_link_directories(foo PUBLIC ../bar)
target_include_directories(foo PUBLIC ../bar)
#add_subdirectory(./xxx)
$

$ cat src/main1/CMakeLists.txt
#cmake_minimum_required(VERSION 3.24)
#project(myproject VERSION 1.0)
#set(CMAKE_C_FLAGS "-std=c11 -g") # -O3 # .cpp
file(GLOB SOURCES *.c) # .cpp

add_executable(main1 ${SOURCES})
#add_library(main1 SHARED ${SOURCES})
target_link_libraries(main1 PUBLIC foo)
target_link_directories(main1 PUBLIC ../foo)
target_include_directories(main1 PUBLIC ../foo)
#add_subdirectory(./xxx)
$

$ cat src/main2/CMakeLists.txt
#cmake_minimum_required(VERSION 3.24)
#project(myproject VERSION 1.0)
#set(CMAKE_C_FLAGS "-std=c11 -g") # -O3 # .cpp
file(GLOB SOURCES *.c) # .cpp

add_executable(main2 ${SOURCES})
#add_library(main2 SHARED ${SOURCES})
target_link_libraries(main2 PUBLIC foo)
target_link_directories(main2 PUBLIC ../foo)
target_include_directories(main2 PUBLIC ../foo)
#add_subdirectory(./xxx)
$

$ cat .vscode/launch.json
{
  // Use IntelliSense to learn about possible attributes.
  // Hover to view descriptions of existing attributes.
  // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
  "version": "0.2.0",
  "configurations": [
    {
      "type": "lldb",
      "request": "launch",
      "name": "Debug",
      // "program": "${fileDirname}/${fileBasenameNoExtension}", //new
      "program": "${workspaceFolder}/src/build/${fileBasenameNoExtension}/${fileBasenameNoExtension}", //new
      "args": [],
      "cwd": "${workspaceFolder}/src/build/${fileBasenameNoExtension}/",
      "preLaunchTask": "make", //new
    }
  ]
}
$

$ cat .vscode/tasks.json
{
  // See https://go.microsoft.com/fwlink/?LinkId=733558
  // for the documentation about the tasks.json format
  "version": "2.0.0",
  "tasks": [
    {
      "label": "make", //new
      "type": "shell",
      "options": {
        "cwd": "${workspaceFolder}/src",
      },
      "command": "cmake --build build", //new
    }
  ]
}
$

$ cat .vscode/settings.json
{
  "clangd.fallbackFlags": [
    // "-std=c++2a",
    // "-I${workspaceFolder}/boost",
    "-I${workspaceFolder}/src/foo",
    "-I${workspaceFolder}/src/bar",
  ],
}
$

$ cd src

$ cmake -S . -B build
$ cmake --build build
$

# or,
$ mkdir build && cd build
$ cmake ..
$ make
