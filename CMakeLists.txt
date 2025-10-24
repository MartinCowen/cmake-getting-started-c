# CMakeLists.txt for C++ projects
# Most of this is from Jason Turner's cpp_starter_project described at https://youtu.be/YbgH7yat-Jo
# The original has changed to https://github.com/cpp-best-practices/cmake_template so see https://github.com/AnthonyUtt/cpp_default_project for the original
# Another good one is at https://github.com/aminya/project_options

cmake_minimum_required(VERSION 4.0.0)

# Project name and language, eg C or CXX for C++
project(hello C)
set(CMAKE_C_STANDARD 99)    # Sets the compiler standard used

# target_include_directories(main PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/include/")
file(GLOB_RECURSE C_SOURCES CONFIGURE_DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/*.c")
list(FILTER C_SOURCES EXCLUDE REGEX "^${CMAKE_CURRENT_SOURCE_DIR}/build/")

include(CMakePrintHelpers)
cmake_print_variables(C_SOURCES)

add_executable(hello "${C_SOURCES}")

include(cmake/StandardProjectSettings.cmake)
include(cmake/PreventInSourceBuilds.cmake)

# Link this 'library' to set the c standard / compile-time options requested
add_library(project_options INTERFACE)
target_compile_features(project_options INTERFACE c_std_99)

if(CMAKE_C_COMPILER_ID MATCHES ".*Clang")
  option(ENABLE_BUILD_WITH_TIME_TRACE "Enable -ftime-trace to generate time tracing .json files on clang" OFF)
  if(ENABLE_BUILD_WITH_TIME_TRACE)
    add_compile_definitions(project_options INTERFACE -ftime-trace)
  endif()
endif()

# Link this 'library' to use the warnings specified in CompilerWarnings.cmake
add_library(project_warnings INTERFACE)

# enable cache system
option(ENABLE_CMAKE_CACHE "Enable CMake Cache to speed up" OFF)
if(ENABLE_CMAKE_CACHE)
    include(cmake/Cache.cmake)
endif()

# standard compiler warnings
include(cmake/CompilerWarnings.cmake)
set_project_warnings(project_warnings)

# sanitizer options if supported by compiler
include(cmake/Sanitizers.cmake)
enable_sanitizers(project_options)

# enable doxygen
include(cmake/Doxygen.cmake)
enable_doxygen()

# allow for static analysis options
include(cmake/StaticAnalyzers.cmake)

# CMake Test system
option(ENABLE_CTEST "Enable CMake Tests" OFF)

if (ENABLE_CTEST)
    include(CTest)
    enable_testing()
endif()
