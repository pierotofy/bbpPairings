cmake_minimum_required(VERSION 3.16)

project(swisser 
    VERSION 1.0.0
    DESCRIPTION "BBP Pairings - Swiss System Tournament Pairing Software"
    LANGUAGES CXX
)

# Set C++ standard
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Configuration options (matching Makefile flags)
option(ENABLE_BURSTEIN "Enable Burstein Swiss system" OFF)
option(ENABLE_DUTCH "Enable Dutch Swiss system" ON)
option(ENABLE_ENGINE_COMPARISON "Enable tournament checker and random tournament generator" OFF)
option(ENABLE_DEBUG "Enable debug build" OFF)
option(ENABLE_PROFILE "Enable profile build" OFF)
option(ENABLE_OPTIMIZE "Enable optimization" ON)

# Optional configuration parameters
set(MAX_PLAYERS "" CACHE STRING "Maximum number of players (default: 9999)")
set(MAX_POINTS "" CACHE STRING "Maximum points (default: 999)")
set(MAX_RATING "" CACHE STRING "Maximum rating (default: 9999)")
set(MAX_ROUNDS "" CACHE STRING "Maximum rounds (default: 99)")

# Get version information from git
find_package(Git QUIET)
if(Git_FOUND)
    execute_process(
        COMMAND ${GIT_EXECUTABLE} describe --exact-match
        WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
        OUTPUT_VARIABLE GIT_VERSION
        ERROR_QUIET
        OUTPUT_STRIP_TRAILING_WHITESPACE
    )
    
    if(NOT GIT_VERSION)
        execute_process(
            COMMAND ${GIT_EXECUTABLE} describe
            WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
            OUTPUT_VARIABLE GIT_VERSION
            ERROR_QUIET
            OUTPUT_STRIP_TRAILING_WHITESPACE
        )
        if(GIT_VERSION)
            set(VERSION_INFO "non-release build ${GIT_VERSION}")
        else()
            set(VERSION_INFO "non-release build")
        endif()
    else()
        set(VERSION_INFO "${GIT_VERSION}")
    endif()
else()
    set(VERSION_INFO "unknown version")
endif()

# Create the executable
#add_executable(bbpPairings ${SOURCES})

# Define source files
set(SOURCES src/swisser.cpp 
            src/swisssystems/common.cpp 
            src/matching/computer.cpp
            src/matching/computer.cpp
            src/matching/detail/graph.cpp
            src/matching/detail/parentblossom.cpp
            src/matching/detail/rootblossom.cpp)

if(ENABLE_DUTCH)
    list(APPEND SOURCES src/swisssystems/dutch.cpp)
endif()
add_executable(swisser ${SOURCES})

# Set output name to match Makefile
set_target_properties(swisser PROPERTIES OUTPUT_NAME "swisser")

# Include directories
target_include_directories(swisser PRIVATE src)

# Compiler flags (Linux/GCC specific, ignoring Windows flags)
target_compile_options(swisser PRIVATE
    -ftabstop=2
    -Werror
    -Wfatal-errors
    -pedantic
    -pedantic-errors
    -Wall
    -Walloca
    -Wcast-qual
    -Wctor-dtor-privacy
    -Wdisabled-optimization
    -Wdouble-promotion
    -Wextra
    -Wextra-semi
    -Wformat=2
    -Winvalid-pch
    -Wmismatched-tags
    -Wmissing-braces
    -Wmissing-declarations
    -Wmissing-include-dirs
    -Wnull-dereference
    -Woverloaded-virtual
    -Wpacked
    -Wpointer-arith
    -Wredundant-decls
    -Wsign-promo
    -Wstrict-overflow=4
    -Wsuggest-override
    -Wundef
    -Wunknown-pragmas
    -Wunused-macros
    -Wvla
    -Wzero-as-null-pointer-constant
    -Wno-free-nonheap-object
    -Wno-overflow
    -Wno-sign-compare
)

# GCC-specific flags (assuming GCC on Linux)
if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    target_compile_options(swisser PRIVATE
        -Wabi=0
        -Waligned-new=all
        -Walloc-zero
        -Warray-bounds=2
        -Wattribute-alias=2
        -Wcatch-value=3
        -Wconditionally-supported
        -Wduplicated-cond
        -Wformat-overflow=2
        -Wformat-signedness
        -Wformat-truncation=2
        -Wimplicit-fallthrough=3
        -Wlogical-op
        -Wmultiple-inheritance
        -Wnormalized=nfkc
        -Wplacement-new=2
        -Wshadow=local
        -Wstrict-null-sentinel
        -Wstringop-overflow=4
        -Wsuggest-attribute=cold
        -Wsuggest-attribute=format
        -Wsuggest-attribute=malloc
        -Wsuggest-attribute=noreturn
        -Wsuggest-final-methods
        -Wsuggest-final-types
        -Wsync-nand
        -Wtrampolines
        -Wunsafe-loop-optimizations
        -Wvector-operation-performance
        -Wvirtual-inheritance
        -Wno-maybe-uninitialized
        -Wno-error=deprecated-declarations
    )
    
    # GCC version-specific flags
    if(CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL "11.0")
        target_compile_options(swisser PRIVATE
            -Warray-parameter=2
            -Wctad-maybe-unsupported
            -Wenum-conversion
            -Winvalid-imported-macros
        )
    endif()
endif()

# Build type specific flags
if(ENABLE_DEBUG)
    target_compile_options(swisser PRIVATE -ggdb)
    if(ENABLE_OPTIMIZE)
        target_compile_options(swisser PRIVATE -Og)
    endif()
else()
    target_compile_definitions(swisser PRIVATE NDEBUG)
    target_link_options(swisser PRIVATE -s)
    if(ENABLE_OPTIMIZE)
        target_compile_options(swisser PRIVATE -O3)
    endif()
endif()

if(ENABLE_PROFILE)
    target_compile_options(swisser PRIVATE -ggdb -O3)
    if(NOT ENABLE_DEBUG)
        target_compile_definitions(swisser PRIVATE NDEBUG)
    endif()
endif()

# Link Time Optimization
if(ENABLE_OPTIMIZE AND CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    target_compile_options(swisser PRIVATE -flto)
    target_link_options(swisser PRIVATE -flto)
endif()

# Conditional compilation flags based on enabled features
if(NOT ENABLE_BURSTEIN)
    target_compile_definitions(swisser PRIVATE OMIT_BURSTEIN)
endif()

if(NOT ENABLE_DUTCH)
    target_compile_definitions(swisser PRIVATE OMIT_DUTCH)
endif()

if(NOT ENABLE_ENGINE_COMPARISON)
    target_compile_definitions(swisser PRIVATE OMIT_GENERATOR OMIT_CHECKER)
endif()

# Optional maximum value definitions
if(MAX_PLAYERS)
    target_compile_definitions(swisser PRIVATE MAX_PLAYERS=${MAX_PLAYERS})
endif()

if(MAX_POINTS)
    target_compile_definitions(swisser PRIVATE MAX_POINTS=${MAX_POINTS})
endif()

if(MAX_RATING)
    target_compile_definitions(swisser PRIVATE MAX_RATING=${MAX_RATING})
endif()

if(MAX_ROUNDS)
    target_compile_definitions(swisser PRIVATE MAX_ROUNDS=${MAX_ROUNDS})
endif()

# Version information
target_compile_definitions(swisser PRIVATE VERSION_INFO="${VERSION_INFO}")

# Test support
enable_testing()

# Add test subdirectory if it exists
if(EXISTS "${CMAKE_SOURCE_DIR}/test/CMakeLists.txt")
    add_subdirectory(test)
elseif(EXISTS "${CMAKE_SOURCE_DIR}/test")
    # Create a simple test that runs the main executable
    add_test(
        NAME basic_test
        COMMAND $<TARGET_FILE:swisser> -r
        WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
    )
endif()

# Installation
install(TARGETS swisser
    RUNTIME DESTINATION bin
)

install(FILES README.txt LICENSE.txt Apache-2.0.txt
    DESTINATION share/doc/swisser
    OPTIONAL
)

# Print configuration summary
message(STATUS "BBP Pairings Configuration:")
message(STATUS "  Version: ${VERSION_INFO}")
message(STATUS "  C++ Standard: ${CMAKE_CXX_STANDARD}")
message(STATUS "  Burstein Swiss System: ${ENABLE_BURSTEIN}")
message(STATUS "  Dutch Swiss System: ${ENABLE_DUTCH}")
message(STATUS "  Engine Comparison: ${ENABLE_ENGINE_COMPARISON}")
message(STATUS "  Debug: ${ENABLE_DEBUG}")
message(STATUS "  Profile: ${ENABLE_PROFILE}")
message(STATUS "  Optimize: ${ENABLE_OPTIMIZE}")
if(MAX_PLAYERS)
    message(STATUS "  Max Players: ${MAX_PLAYERS}")
endif()
if(MAX_POINTS)
    message(STATUS "  Max Points: ${MAX_POINTS}")
endif()
if(MAX_RATING)
    message(STATUS "  Max Rating: ${MAX_RATING}")
endif()
if(MAX_ROUNDS)
    message(STATUS "  Max Rounds: ${MAX_ROUNDS}")
endif()
