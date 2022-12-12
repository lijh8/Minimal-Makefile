cmake_minimum_required(VERSION 3.24)
project(myproject VERSION 1.0)
set(CMAKE_C_FLAGS "-std=c2x -g") # -O3 # .cpp
file(GLOB SOURCES *.c) # .cpp

add_executable(main ${SOURCES})
target_include_directories(main PUBLIC ./hello)
target_link_directories(main PUBLIC ./hello)
target_link_libraries(main PUBLIC hello)
#add_subdirectory(./hello)
