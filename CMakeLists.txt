# CMake configuration for IJG libjpeg
# To install, copy this file and jconfig.h.in into a libjpeg source directory
# Adapted from LuaDist's CMakeLists
# https://github.com/LuaDist/libjpeg/blob/master/CMakeLists.txt

cmake_minimum_required(VERSION 3.0)

### Setup the project ###
file(READ "configure.ac" ac)
string(REGEX MATCH "AC_INIT\\(\\[libjpeg\\],\ \\[([0-9]*\\.[0-9]*\\.[0-9]*)\\]\\)" _ ${ac})
set(version ${CMAKE_MATCH_1})
project(libjpeg VERSION ${version} LANGUAGES C)
set(C_STANDARD 99)

### Include extra packages ###
include(CMakeDependentOption)
include(GNUInstallDirs)

### Options ###
option(BUILD_SHARED_LIBS "Build shared libraries" ON)
option(BUILD_STATIC_LIBS "Build static libraries" ON)
cmake_dependent_option(BUILD_EXECUTABLES "Build JPEG utilities" ON "BUILD_SHARED_LIBS OR BUILD_STATIC_LIBS" OFF)
cmake_dependent_option(BUILD_TESTS "Build test executables" ON "BUILD_SHARED_LIBS OR BUILD_STATIC_LIBS;BUILD_EXECUTABLES" OFF)
cmake_dependent_option(LINK_STATIC "Link all executables statically" OFF "BUILD_STATIC_LIBS;BUILD_EXECUTABLES" OFF)
cmake_dependent_option(BUILD_ALT_UI "Build alternate interface for command line utilities" OFF "BUILD_EXECUTABLES" OFF)

# Make sure we build at least one library
if(NOT(BUILD_SHARED_LIBS OR BUILD_STATIC_LIBS))
  message(FATAL_ERROR "Both static and shared libraries are disabled. Nothing will be built.")
endif()

### Configure jconfig.h ###
include(ConfigureJConfig.cmake)

### Build the object library ###
set(PUBLIC_HDRS
  jconfig.h
  jerror.h
  jmorecfg.h
  jpeglib.h
)

set(SRCS
  jaricom.c jcapimin.c jcapistd.c jcarith.c jccoefct.c jccolor.c
  jcdctmgr.c jchuff.c jcinit.c jcmainct.c jcmarker.c jcmaster.c jcomapi.c
  jcparam.c jcprepct.c jcsample.c jctrans.c jdapimin.c jdapistd.c jdarith.c
  jdatadst.c jdatasrc.c jdcoefct.c jdcolor.c jddctmgr.c jdhuff.c jdinput.c
  jdmainct.c jdmarker.c jdmaster.c jdmerge.c jdpostct.c jdsample.c jdtrans.c
  jerror.c jfdctflt.c jfdctfst.c jfdctint.c jidctflt.c jidctfst.c jidctint.c
  jmemmgr.c jmemnobs.c jquant1.c jquant2.c jutils.c
)

add_library(jpeg STATIC ${SRCS})
target_include_directories(jpeg
  PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
    $<INSTALL_INTERFACE:include>
)
set_target_properties(jpeg PROPERTIES POSITION_INDEPENDENT_CODE ON)
