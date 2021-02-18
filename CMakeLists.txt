# Copyright (C) 2020 - <---@gmail.com>
# This file is subject to the license terms in the LICENSE file
# found in the top-level directory of this distribution.

cmake_minimum_required(VERSION 3.0)
project(pointcloud VERSION 1.2.1)

set (POINTCLOUD_VERSION_MAJOR 1)
set (POINTCLOUD_VERSION_MINOR 2)
set (POINTCLOUD_VERSION_PATCH 1)

set (CMAKE_COLOR_MAKEFILE ON)

# Path to additional CMake modules
set(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake/modules ${CMAKE_MODULE_PATH})

if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE RelWithDebInfo CACHE STRING 
        "Choose the type of build, options are: None Debug Release RelWithDebInfo MinSizeRel" FORCE)
endif()
message(STATUS "Setting ${CMAKE_PROJECT_NAME} build type - ${CMAKE_BUILD_TYPE}")

#------------------------------------------------------------------------------
# check for headers

include (CheckIncludeFiles)

check_include_files (stdint.h HAVE_STDINT_H)
check_include_files (getopt.h HAVE_GETOPT_H)

#------------------------------------------------------------------------------
# PostgreSQL server paths come from pg_config
#
find_package(PostgreSQL CONFIG REQUIRED)
if (PostgreSQL_FOUND)
  message(STATUS "find_package PostgreSQL:")
  message(STATUS ${PostgreSQL_LIBRARIES})
  message(STATUS ${PostgreSQL_INCLUDE_DIRS})
  message(STATUS ${PostgreSQL_LIBRARY_DIRS})
  message(STATUS ${PostgreSQL_VERSION_STRING})
endif()

#------------------------------------------------------------------------------
# libxml2

find_package (LibXml2 REQUIRED)
mark_as_advanced (CLEAR LIBXML2_INCLUDE_DIR)
mark_as_advanced (CLEAR LIBXML2_LIBRARIES)
include_directories (${LIBXML2_INCLUDE_DIR})


#------------------------------------------------------------------------------
# zlib

find_package (ZLIB REQUIRED)
include_directories (${ZLIB_INCLUDE_DIR})


#------------------------------------------------------------------------------
# cunit and ght

find_package (CUnit)
find_package (LibGHT)

if (LIBGHT_FOUND)
  set (HAVE_LIBGHT 1)
endif (LIBGHT_FOUND)

#------------------------------------------------------------------------------
# use default install location if not specified

if (NOT DEFINED LIB_INSTALL_DIR)
  set (LIB_INSTALL_DIR lib)
endif()


#------------------------------------------------------------------------------
# generate config include

configure_file (
  "${PROJECT_SOURCE_DIR}/lib/pc_config.h.cmake"
  "${PROJECT_BINARY_DIR}/lib/pc_config.h"
  )

include_directories ("${PROJECT_BINARY_DIR}/lib")


#------------------------------------------------------------------------------
# libpc is required for the database component

add_subdirectory (lib)
add_subdirectory (pgsql)
add_subdirectory (pgsql_postgis)

#add_subdirectory(lib)
#add_executable(pointcloud lib\lazperf_adapter.cpp)
#add_executable(pointcloud lib\hashtable.c)
#add_executable(pointcloud lib\pc_bytes.c)
#add_executable(pointcloud lib\pc_dimstats.c)
#add_executable(pointcloud lib\pc_filter.c)
#add_executable(pointcloud lib\pc_mem.c)
#add_executable(pointcloud lib\pc_patch.c)
#add_executable(pointcloud lib\pc_patch_dimensional.c)
#add_executable(pointcloud lib\pc_patch_lazperf.c)
#add_executable(pointcloud lib\pc_patch_uncompressed.c)
#add_executable(pointcloud lib\pc_point.c)
#add_executable(pointcloud lib\pc_pointlist.c)
#add_executable(pointcloud lib\pc_schema.c)
#add_executable(pointcloud lib\pc_sort.c)
#add_executable(pointcloud lib\pc_stats.c)
#add_executable(pointcloud lib\pc_util.c)
#add_executable(pointcloud lib\pc_val.c)
#add_executable(pointcloud lib\stringbuffer.c)
##unittest
##add_executable(pointcloud lib\cunit\cu_pc_bytes.c)
##add_executable(pointcloud lib\cunit\cu_pc_patch.c)
##add_executable(pointcloud lib\cunit\cu_pc_patch_lazperf.c)
##add_executable(pointcloud lib\cunit\cu_pc_point.c)
##add_executable(pointcloud lib\cunit\cu_pc_schema.c)
##add_executable(pointcloud lib\cunit\cu_pc_sort.c)
##add_executable(pointcloud lib\cunit\cu_pc_util.c)
##add_executable(pointcloud lib\cunit\cu_tester.c)
##add_subdirectory(pgsql)
#add_executable(pointcloud pgsql\pc_access.c)
#add_executable(pointcloud pgsql\pc_editor.c)
#add_executable(pointcloud pgsql\pc_inout.c)
#add_executable(pointcloud pgsql\pc_pgsql.c)

target_include_directories(pointcloud PUBLIC ${PostgreSQL_INCLUDE_DIRS})
target_link_libraries(pointcloud PUBLIC PostgreSQL::PostgreSQL)

# set_target_properties(pointcloud PROPERTIES CXX_STANDARD 11 OUTPUT_NAME pointcloud)

