# This file has been adapted from dynarmic

cmake_minimum_required(VERSION 3.18)
project(sirit CXX)

# Determine if we're built as a subproject (using add_subdirectory)
# or if this is the master project.
set(MASTER_PROJECT OFF)
if (CMAKE_CURRENT_SOURCE_DIR STREQUAL CMAKE_SOURCE_DIR)
  set(MASTER_PROJECT ON)
endif()

# Sirit project options
option(SIRIT_TESTS "Build tests" OFF)
option(SIRIT_USE_SYSTEM_SPIRV_HEADERS "Use system SPIR-V headers" OFF)

# Default to a Release build
if (NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE "Release" CACHE STRING "Choose the type of build, options are: Debug Release RelWithDebInfo MinSizeRel." FORCE)
    message(STATUS "Defaulting to a Release build")
endif()

# Set hard requirements for C++
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Warn on CMake API deprecations
set(CMAKE_WARN_DEPRECATED ON)

# Disable in-source builds
set(CMAKE_DISABLE_SOURCE_CHANGES ON)
set(CMAKE_DISABLE_IN_SOURCE_BUILD ON)
if ("${CMAKE_SOURCE_DIR}" STREQUAL "${CMAKE_BINARY_DIR}")
    message(SEND_ERROR "In-source builds are not allowed.")
endif()

# Add the module directory to the list of paths
list(APPEND CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/CMakeModules")

# Set compiler ID
if (CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
  set(msvc TRUE)
elseif (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
  set(gcc TRUE)
elseif (CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
  set(clang TRUE)
endif()

# Compiler flags
if (msvc AND NOT clang)
    set(SIRIT_CXX_FLAGS
        /std:c++latest # CMAKE_CXX_STANDARD as no effect on MSVC until CMake 3.10.
        /W4
        /w34263 # Non-virtual member function hides base class virtual function
        /w44265 # Class has virtual functions, but destructor is not virtual
        /w34456 # Declaration of 'var' hides previous local declaration
        /w34457 # Declaration of 'var' hides function parameter
        /w34458 # Declaration of 'var' hides class member
        /w34459 # Declaration of 'var' hides global definition
        /w34946 # Reinterpret-cast between related types
        /wd4592 # Symbol will be dynamically initialized (implementation limitation)
        /permissive- # Stricter C++ standards conformance
        /MP
        /Zi
        /Zo
        /EHsc
        /Zc:inline # Omits inline functions from object-file output
        /DNOMINMAX)

    if (CMAKE_VS_PLATFORM_TOOLSET MATCHES "LLVM-vs[0-9]+")
        list(APPEND SIRIT_CXX_FLAGS
             -Qunused-arguments
             -Wno-missing-braces)
    endif()
else()
    set(SIRIT_CXX_FLAGS
        -Wall
        -Wextra
        -Wpedantic
        -Wno-missing-braces
        -Wno-unused-parameter
        -Wno-unknown-pragmas
        -Wno-implicit-fallthrough
        -Wno-unused-variable
        -Wno-c++98-compat
        -Wno-pre-c++14-compa
        -Wno-pre-c++17-compat
        -Wno-sign-compare)
endif()

# Enable unit-testing.
enable_testing(true)

# SPIR-V headers
if (SIRIT_USE_SYSTEM_SPIRV_HEADERS)
    find_package(SPIRV-Headers REQUIRED)
else()
    add_subdirectory(externals/SPIRV-Headers EXCLUDE_FROM_ALL)
endif()

# Sirit project files
add_subdirectory(src)
if (SIRIT_TESTS)
    add_subdirectory(tests)
endif()
