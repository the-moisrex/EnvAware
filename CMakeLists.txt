cmake_minimum_required(VERSION 3.23...3.29)

project(envaware
        VERSION 1.0.0
        LANGUAGES CXX
)
enable_language(CXX)
set(LIB_NAME ${PROJECT_NAME})
set(LIB_SOVERSION 1)
set(BIN_SOVERSION 1)
set(LIB_VERSION 1.0.0)
set(BIN_VERSION 1.0.0)

# output the libraries and executables in the cmake home directory instead of
# their own folders
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR})
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR})
set(CMAKE_PREFIX_PATH "${CMAKE_PREFIX_PATH};${CMAKE_BINARY_DIR}/envaware")
set(LIB_DIR "${CMAKE_CURRENT_SOURCE_DIR}/envaware")
set(LIB_INCLUDE_DIR "${LIB_DIR}")
set(TESTS_DIR "${CMAKE_CURRENT_SOURCE_DIR}/tests")
set(INSTALL_LIBDIR $<IF:$<BOOL:${CMAKE_INSTALL_LIBDIR}>,${CMAKE_INSTALL_LIBDIR},lib>)
set(INSTALL_BINDIR $<IF:$<BOOL:${CMAKE_INSTALL_BINDIR}>,${CMAKE_INSTALL_BINDIR},bin>)
set(INSTALL_INCLUDEDIR $<IF:$<BOOL:${CMAKE_INSTALL_INCLUDEDIR}>,${CMAKE_INSTALL_INCLUDEDIR},include>)
set(INSTALL_ARCHIVEDIR ${INSTALL_LIBDIR})
set(gcc_min_version 12.1.0)
set(clang_min_version 14.0.6)

if (CMAKE_BUILD_TYPE STREQUAL "Debug")
    set(IS_DEBUG TRUE)
    message(STATUS "Debug mode is enabled.")
else ()
    set(IS_DEBUG FALSE)
    message(STATUS "Debug mode is disabled.")
endif ()

if (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    set(IS_GCC TRUE)
    set(IS_CLANG FALSE)
    if (CMAKE_CXX_COMPILER_VERSION VERSION_LESS ${gcc_min_version})
        message(FATAL_ERROR "This version of GCC is not supported. We need at least version ${gcc_min_version}.")
    endif ()
elseif (CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
    set(IS_GCC FALSE)
    set(IS_CLANG TRUE)
    if (CMAKE_CXX_COMPILER_VERSION VERSION_LESS ${clang_min_version})
        message(FATAL_ERROR "This version of Clang is not supported. We need at least version ${clang_min_version}.")
    endif ()
endif ()

#option(SHARED_LIBRARY_EXECUTABLE "Make the shared library executable." OFF)
option(NATIVE_ARCH "Optimize the binaries for this system architecture only." OFF)
option(VERBOSE "Verbose printing" OFF)


if (IS_CLANG AND IS_DEBUG)
    option(FUZZ_TESTS "Enable Building of fuzz tests" ON)
else ()
    option(FUZZ_TESTS "Enable Building of fuzz tests" OFF)
endif ()

message(STATUS "CMake Version                  : ${CMAKE_VERSION}")
message(STATUS "Project name                   : ${PROJECT_NAME}")
message(STATUS "C++ Standard Version           : C++${CMAKE_CXX_STANDARD}")
message(STATUS "Library version                : ${LIB_VERSION}")
message(STATUS "Library soversion              : ${LIB_SOVERSION}")
message(STATUS "Binary version                 : ${BIN_VERSION}")
message(STATUS "Binary soversion               : ${BIN_SOVERSION}")
message(STATUS "Library name                   : ${LIB_NAME}")
message(STATUS "EXPORT_NAME                    : ${EXPORT_NAME}")
message(STATUS "Compiler                       : ${CMAKE_CXX_COMPILER_ID} ${CMAKE_CXX_COMPILER_VERSION} (${CMAKE_CXX_COMPILER})")
message(STATUS "prefix path                    : ${CMAKE_PREFIX_PATH}")
message(STATUS "Library path                   : ${LIB_DIR}")
message(STATUS "Library include path           : ${LIB_INCLUDE_DIR}")
message(STATUS "Tests path                     : ${TESTS_DIR}")
message(STATUS "Install lib dir expression     : ${INSTALL_LIBDIR}")
message(STATUS "Install archive dir expression : ${INSTALL_ARCHIVEDIR}")
message(STATUS "Install bin dir expression     : ${INSTALL_BINDIR}")
message(STATUS "Install include dir expression : ${INSTALL_INCLUDEDIR}")
message(STATUS "Building Fuzz Tests            : ${FUZZ_TESTS}")


add_compile_definitions(envaware_PROJECT_ROOT=${CMAKE_CURRENT_SOURCE_DIR})

if (IS_DEBUG)
    enable_testing()
endif ()

if (IS_GCC AND IS_DEBUG)
    set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -g3 -fconcepts-diagnostics-depth=10 -ftemplate-backtrace-limit=0 -Wall -Wpedantic -Wextra -Wconversion -Wshadow")

    # For profiling (https://www.jetbrains.com/help/clion/cpu-profiler.html#UsingTheProfiler)
    set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -O0 -fno-omit-frame-pointer")

    # set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -ftime-report")
elseif (IS_CLANG AND IS_DEBUG)
    set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -g3 -Wall -Wpedantic -Wextra -Wconversion -Wshadow")

    # For profiling (https://www.jetbrains.com/help/clion/cpu-profiler.html#UsingTheProfiler)
    set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -O0 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer")

    # set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -ftime-trace")
endif ()


if (IS_CLANG)
    set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
endif ()


option(FORCE_COLORED_OUTPUT "Always produce ANSI-colored output (GNU/Clang only)." TRUE)

if (${FORCE_COLORED_OUTPUT})
    if (IS_GCC)
        add_compile_options(-fdiagnostics-color=always)
    elseif (IS_CLANG)
        add_compile_options(-fcolor-diagnostics)
    endif ()
endif ()


add_subdirectory(${LIB_DIR})

if (IS_DEBUG)
    add_subdirectory(${TESTS_DIR} EXCLUDE_FROM_ALL)
endif ()
