cmake_minimum_required(VERSION 3.20)
project(GUILib LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/bin)

find_package(SDL3 CONFIG REQUIRED)

# Header-only library target
add_library(GUILib INTERFACE)
target_include_directories(GUILib INTERFACE
    $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/include>
    $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/external>
    $<INSTALL_INTERFACE:include>
)
target_link_libraries(GUILib INTERFACE SDL3::SDL3)
target_compile_features(GUILib INTERFACE cxx_std_23)

# Test executable
add_executable(main-test test/main-test.cpp)
target_link_libraries(main-test PRIVATE GUILib SDL3::SDL3main)
