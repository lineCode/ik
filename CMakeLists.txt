cmake_minimum_required (VERSION 3.0)
cmake_policy (SET CMP0054 NEW)

# If the user specifies -DCMAKE_BUILD_TYPE on the command line, take their
# definition and dump it in the cache along with proper documentation, otherwise
# set MAKE_BUILD_TYPE to Debug prior to calling PROJECT()
if (DEFINED CMAKE_BUILD_TYPE)
    set (CMAKE_BUILD_TYPE ${CMAKE_BUILD_TYPE} CACHE STRING "Choose the type of build, options are: None(CMAKE_CXX_FLAGS or CMAKE_C_FLAGS used) Debug Release RelWithDebInfo MinSizeRel.")
else()
    set (CMAKE_BUILD_TYPE "Debug" CACHE STRING "Choose the type of build, options are: None(CMAKE_CXX_FLAGS or CMAKE_C_FLAGS used) Debug Release RelWithDebInfo MinSizeRel.")
endif()
message (STATUS "Build type: ${CMAKE_BUILD_TYPE}")

###############################################################################
# Project name
###############################################################################

project ("ik" C)

# Make macs happy
set (CMAKE_MACOSX_RPATH OFF)

###############################################################################
# output locations
###############################################################################

set (CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib")
set (CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin")
if (WIN32)
    set (CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin")
else ()
    set (CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib")
endif ()

set (IK_RPATHS
    "../lib")

###############################################################################
# compiler definitions and flags
###############################################################################

if (${CMAKE_C_COMPILER_ID} STREQUAL "GNU")
    add_definitions (-W -Wall -fno-omit-frame-pointer)
elseif (${CMAKE_C_COMPILER_ID} STREQUAL "Clang")
    add_definitions (-W -Wall -fno-omit-frame-pointer)
elseif (${CMAKE_C_COMPILER_ID} STREQUAL "Intel")
elseif (${CMAKE_C_COMPILER_ID} STREQUAL "MSVC")
    add_definitions (-D_CRT_SECURE_NO_WARNINGS)
    if (CMAKE_BUILD_TYPE MATCHES Debug)
        STRING(REGEX REPLACE "/MDd" "" CMAKE_C_FLAGS_DEBUG ${CMAKE_C_FLAGS_DEBUG})
        add_definitions (-MTd)
    elseif (CMAKE_BUILD_TYPE MATCHES Release)
        STRING(REGEX REPLACE "/MD" "" CMAKE_C_FLAGS_RELEASE ${CMAKE_C_FLAGS_RELEASE})
        add_definitions (-MT)
    endif ()
endif ()

if (CMAKE_BUILD_TYPE MATCHES Debug)
    add_definitions(-DDEBUG)
endif ()

###############################################################################
# Dependency settings
###############################################################################

message (STATUS "------------------------------------------------------------")
message (STATUS "Global settings")
message (STATUS "------------------------------------------------------------")

add_subdirectory ("ik")

option (IK_TESTS "Whether to build unit tests or not (requires C++)" OFF)
option (IK_BENCHMARKS "Whether to build benchmark tests or not (requires C++)" OFF)
if (IK_TESTS)
    add_subdirectory ("tests")
    set_target_properties (iktests
        PROPERTIES
        INSTALL_RPATH ${IK_RPATHS})
endif ()
if (IK_BENCHMARKS)
    add_subdirectory ("benchmarks")
    set_target_properties (ikbenchmarks
        PROPERTIES
        INSTALL_RPATH ${IK_RPATHS})
endif ()
