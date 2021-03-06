cmake_minimum_required(VERSION 3.15)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

##
## PROJECT
## name and version
##
project(mhwd
        VERSION 0.6.11
        LANGUAGES C CXX)

##
## INCLUDE
##
include(GNUInstallDirs)
include(StandardProjectSettings)
include(CompilerWarnings)
include(EnableCcache)
include(Linker)
include(StaticAnalyzers)
include(Sanitizers)
include(CPM)

CPMAddPackage(
  NAME fmt
  GITHUB_REPOSITORY fmtlib/fmt
  GIT_TAG 7e4ad40171aa552d38cb99a5c181a0d7b150facc
  EXCLUDE_FROM_ALL YES
)

CPMAddPackage(
  NAME hwinfo
  GITHUB_REPOSITORY vnepogodin/hwinfo
  GIT_TAG master
  EXCLUDE_FROM_ALL YES
)

CPMAddPackage(
  NAME x86emu
  GITHUB_REPOSITORY vnepogodin/libx86emu
  GIT_TAG master
  EXCLUDE_FROM_ALL YES
)

if(CMAKE_CXX_COMPILER_ID MATCHES ".*Clang")
   CPMAddPackage(
     NAME range-v3
     GITHUB_REPOSITORY ericniebler/range-v3
     GIT_TAG c7ad677c7fe7a3e94d2a44e2ebe89047c96eeb94
     EXCLUDE_FROM_ALL YES
   )
endif()

##
## CONFIGURATION
##
set(EXECUTABLE_OUTPUT_PATH ${CMAKE_CURRENT_SOURCE_DIR}/bin)

# Link this 'library' to set the c++ standard / compile-time options requested
add_library(project_options INTERFACE)
target_compile_features(project_options INTERFACE cxx_std_20)

# Link this 'library' to use the warnings specified in CompilerWarnings.cmake
add_library(project_warnings INTERFACE)
set_project_warnings(project_warnings)

# Add linker configuration
configure_linker(project_options)

# sanitizer options if supported by compiler
enable_sanitizers(project_options)

option(ENABLE_UNITY "Enable Unity builds of projects" OFF)
if(ENABLE_UNITY)
   # Add for any project you want to apply unity builds for
   set_target_properties(${PROJECT_NAME} PROPERTIES UNITY_BUILD ON)
endif()

include_directories(${CMAKE_SOURCE_DIR}/src)

add_subdirectory(libmhwd)
add_subdirectory(src)
add_subdirectory(scripts)
