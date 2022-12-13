cmake_minimum_required(VERSION 3.24)
project(myproject VERSION 1.0)
set(CMAKE_CXX_FLAGS "-std=c++2a -g") # -O3 # .cpp
file(GLOB SOURCES *.cpp) # .cpp

add_executable(main ${SOURCES})
#add_library(hello SHARED ${SOURCES})
#target_link_libraries(main PUBLIC hello)
target_link_directories(main PUBLIC ./hello)
target_include_directories(main PUBLIC ./hello)
#add_subdirectory(./hello)
