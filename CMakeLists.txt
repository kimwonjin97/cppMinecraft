cmake_minimum_required(VERSION 3.16...3.23)

project(cppMinecraft
        VERSION 0.0.1
        DESCRIPTION "minecraft clone"
        HOMEPAGE_URL "%%myurl%%"
        LANGUAGES CXX C)

set(CMAKE_CXX_STANDARD 20)

# strongly encouraged to enable this globally to avoid conflicts between
# -Wpedantic being enabled and -std=c++20 and -std=gnu++20 for example
# when compiling with PCH enabled
set(CMAKE_CXX_EXTENSIONS ON)

set(ENABLE_DEVELOPER_MODE True CACHE BOOL "Enable 'developer mode'")

set(WARNINGS_AS_ERRORS_DEVELOPER_DEFAULT False)
set(ENABLE_CLANG_TIDY_DEVELOPER_DEFAULT False)
set(ENABLE_PCH_DEVELOPER_DEFAULT True)
#set(ENABLE_COVERAGE_DEVELOPER_DEFAULT False)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
include(Index)
include(DynamicProjectOptions)

# run_vcpkg()

set(DEFAULT_CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_LIBRARY_OUTPUT_DIRECTORY})
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/${CMAKE_BUILD_TYPE}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/${CMAKE_BUILD_TYPE}/lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/${CMAKE_BUILD_TYPE}/lib)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# This variable is set by project() in CMake 3.21+
string(
        COMPARE EQUAL
        "${CMAKE_SOURCE_DIR}"
        "${PROJECT_SOURCE_DIR}"
        PROJECT_IS_TOP_LEVEL)
if (PROJECT_IS_TOP_LEVEL)
    # Consider the CTest module, which creates targets and options!
    # Only needed if you want to enable submissions to a CDash server.
    include(CTest)
endif ()

dynamic_project_options(
        # Note: PCH is disabled by default in developer mode because these headers become
        # globally included and they can mask other errors
        PCH_HEADERS
        <vector>
        <iostream>
        <string> # This is a list of headers to pre-compile, here are some common ones
        ENABLE_CONAN
        #         CONAN_OPTIONS    # Extra options to pass to conan
        # MSVC_WARNINGS    # Override the defaults for the MSVC warnings
        # CLANG_WARNINGS   # Override the defaults for the CLANG warnings
        # GCC_WARNINGS     # Override the defaults for the GCC warnings
        CPPCHECK_OPTIONS
        --enable=style,performance,warning,portability
        --inline-suppr
        # We cannot act on a bug/missing feature of cppcheck
        --suppress=cppcheckError
        --suppress=internalAstError
        # if a file does not have an internalAstError, we get an unmatchedSuppression error
        --suppress=unmatchedSuppression
        --suppress=passedByValue
        --suppress=syntaxError
        --inconclusive
)

target_compile_features(project_options INTERFACE cxx_std_${CMAKE_CXX_STANDARD})

add_library(cppMinecraft::project_options INTERFACE IMPORTED)
add_library(cppMinecraft::project_warnings INTERFACE IMPORTED)

add_subdirectory(Minecraft)

option(ENABLE_TESTING "Enable the tests" ${PROJECT_IS_TOP_LEVEL})
#set(ENABLE_TESTING OFF)
if (ENABLE_TESTING)
    enable_testing()
    message(AUTHOR_WARNING "Building Tests. Be sure to check out test/constexpr_tests.cpp for constexpr testing")
    add_subdirectory(test)
endif ()

option(ENABLE_FUZZING "Enable the fuzz tests" OFF)
if(ENABLE_FUZZING)
    if(NOT ${CMAKE_CXX_COMPILER_ID} MATCHES AppleClang)
        message(AUTHOR_WARNING "Building Fuzz Tests, using fuzzing sanitizer https://www.llvm.org/docs/LibFuzzer.html")
        add_subdirectory(fuzz_test)
    endif()
endif()

if (MSVC)
    get_all_installable_targets(all_targets)
    message("all_targets=${all_targets}")
    set_target_properties(${all_targets} PROPERTIES VS_DEBUGGER_ENVIRONMENT "PATH=$(VC_ExecutablePath_x64);%PATH%")
endif ()

# set the startup project for the "play" button in MSVC
set_property(DIRECTORY PROPERTY VS_STARTUP_PROJECT Minecraft)

if (CMAKE_SKIP_INSTALL_RULES)
    return()
elseif (NOT PROJECT_IS_TOP_LEVEL)
    return()
endif ()

message(STATUS "============= Build Type: ${CMAKE_BUILD_TYPE}  =============")
message(STATUS "============= Compiler: ${CMAKE_CXX_COMPILER_ID} =============")
message(STATUS "============= Conan Enabled: ${ProjectOptions_ENABLE_CONAN}  =============")
message(STATUS "============= PCH Enabled: ${ProjectOptions_ENABLE_PCH}  =============")
message(STATUS "============= Clang-tidy Enabled: ${ProjectOptions_ENABLE_CLANG_TIDY}  =============" )
