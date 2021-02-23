# Copyright (C) 2020 - <---@gmail.com>
# This file is subject to the license terms in the LICENSE file
# found in the top-level directory of this distribution.
cmake_minimum_required(VERSION 3.0)

#------------------------------------------------------------------------------
# versions
#------------------------------------------------------------------------------
file(READ "Version.config" PGPOINTCLOUD_VERSION)
string(STRIP ${PGPOINTCLOUD_VERSION} PGPOINTCLOUD_VERSION)
set (PGPOINTCLOUD_VERSION "${PGPOINTCLOUD_VERSION}")
string(REGEX REPLACE
       "([0-9]+)\\.([0-9]+)\\.([0-9]+)" "\\1"
       PGPOINTCLOUD_VERSION_MAJOR "${PGPOINTCLOUD_VERSION}")
string(REGEX REPLACE
       "([0-9]+)\\.([0-9]+)\\.([0-9]+)" "\\2"
       PGPOINTCLOUD_VERSION_MINOR "${PGPOINTCLOUD_VERSION}")
string(REGEX REPLACE
       "([0-9]+)\\.([0-9]+)\\.([0-9]+)" "\\3"
       PGPOINTCLOUD_VERSION_PATCH "${PGPOINTCLOUD_VERSION}")
#set (PGPOINTCLOUD_VERSION_MAJOR "${PGPOINTCLOUD_VERSION_MAJOR}")
#set (POINTCLOUD_VERSION_MAJOR 1)
#set (POINTCLOUD_VERSION_MINOR 2)
#set (POINTCLOUD_VERSION_PATCH 1)
message(STATUS ${PGPOINTCLOUD_VERSION})
message(STATUS ${PGPOINTCLOUD_VERSION_MAJOR})
message(STATUS ${PGPOINTCLOUD_VERSION_MINOR})
message(STATUS ${PGPOINTCLOUD_VERSION_PATCH})

project(pointcloud VERSION ${PGPOINTCLOUD_VERSION})

set (CMAKE_COLOR_MAKEFILE ON)

# Path
list(APPEND CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake/modules")

if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE RelWithDebInfo CACHE STRING 
        "Choose the type of build, options are: None Debug Release RelWithDebInfo MinSizeRel" FORCE)
endif()
message(STATUS "Setting ${CMAKE_PROJECT_NAME} build type - ${CMAKE_BUILD_TYPE}")

#------------------------------------------------------------------------------
# check for headers

include (CheckIncludeFiles)

#check_include_files (stdint.h HAVE_STDINT_H)
#check_include_files (getopt.h HAVE_GETOPT_H)

#------------------------------------------------------------------------------
# PostgreSQL server paths come from pg_config
#
find_package(PostgreSQL 12 REQUIRED)
if (PostgreSQL_FOUND)
  message(STATUS "find_package PostgreSQL:")
  message(STATUS ${PostgreSQL_LIBRARIES})
  message(STATUS ${PostgreSQL_INCLUDE_DIRS})
  message(STATUS ${PostgreSQL_LIBRARY_DIRS})
  message(STATUS ${PostgreSQL_VERSION_STRING})
  find_program(PG_CONFIG pg_config)
  if(NOT PG_CONFIG)
    message(FATAL ERROR "Unable to find 'pg_config'")
  endif(NOT PG_CONFIG)

  string(REGEX REPLACE 
    "^PostgreSQL[\t ]+([0-9]+)\\.([0-9]+)\\.([0-9]+)" "\\1.\\2.\\3"
    PGSQL_VERSION "${PostgreSQ+_VERSION_STRING}")

  if(PGSQL_VERSION)
    string(REGEX REPLACE
        "([0-9]+)\\.([0-9]+)\\.([0-9]+)" "\\1"
        PGSQL_VERSION_MAJOR "${PGSQL_VERSION}")
    string(REGEX REPLACE
        "([0-9]+)\\.([0-9]+)\\.([0-9]+)" "\\2"
        PGSQL_VERSION_MINOR "${PGSQL_VERSION}")

    set(PGSQL_NUMERIC_VERSION ${PGSQL_VERSION_MAJOR}${PGSQL_VERSION_MINOR})
  endif()
  message(STATUS ${PGSQL_VERSION_MAJOR})
  message(STATUS ${PGSQL_VERSION_MINOR})

  exec_program(${PG_CONFIG} ARGS --includedir
               OUTPUT_VARIABLE PGSQL_INCLUDEDIR)
  exec_program(${PG_CONFIG} ARGS --includedir-server
               OUTPUT_VARIABLE PGSQL_INCLUDEDIR_SERVER) 
  exec_program(${PG_CONFIG} ARGS --pkglibdir
	       OUTPUT_VARIABLE PGSQL_PKGLIBDIR)
  exec_program(${PG_CONFIG} ARGS --sharedir
	       OUTPUT_VARIABLE PGSQL_SHAREDIR)
  exec_program(${PG_CONFIG} ARGS --bindir
	       OUTPUT_VARIABLE PGSQL_BINDIR)
endif()

#------------------------------------------------------------------------------
# libxml2

find_package (LibXml2 REQUIRED)
if (LibXml2_FOUND)
  #mark_as_advanced (CLEAR LIBXML2_INCLUDE_DIR)
  #mark_as_advanced (CLEAR LIBXML2_LIBRARIES)
  include_directories (${LIBXML2_INCLUDE_DIR})
  # LIBXML2_LIBRARIES
endif()


#------------------------------------------------------------------------------
# zlib

find_package (ZLIB REQUIRED)
include_directories (${ZLIB_INCLUDE_DIR})


#------------------------------------------------------------------------------
# cunit and ght

#find_package (CUnit)
find_package (LibGHT)
if (LIBGHT_FOUND)
  set (HAVE_LIBGHT 1)
endif (LIBGHT_FOUND)

# laz-perf?
# find_package (laz-perf)


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

set(targets_export_name pgpointcloud-targets)
set(PGPOINTCLOUD_CMAKE_DIR "lib/cmake/pgpointcloud" CACHE STRING
    "Installation directory for cmake files, relative to ${CMAKE_INSTALL_PREFIX}.")

# set cmake install default parameters
include(GNUInstallDirs)

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
# target_link_libraries (pointcloud PUBLIC ${PostgreSQL_LIBRARIES})
#target_link_libraries(pointcloud PUBLIC PostgreSQL::PostgreSQL)

# set_target_properties(pointcloud PROPERTIES CXX_STANDARD 11 OUTPUT_NAME pointcloud)

### Install setting ###
#if(PGPOINTCLOUD_ENABLE_INSTALL)
  ### ---[ Create the config.h file
  # set(pgpointcloud_config_h_in "${CMAKE_CURRENT_SOURCE_DIR}/pgpointcloud_config.h.in")
  # set(pgpointcloud_config_h "${CMAKE_CURRENT_BINARY_DIR}/include/pgpointcloud/pgpointcloud_config.h")
  # configure_file("${pgpointcloud_config_h_in}" "${pgpointcloud_config_h}")

  ### ---[ Add the includes subdirectories
  # install(FILES "${pgpointcloud_config_h}" DESTINATION include)
  install(FILES "${PROJECT_BINARY_DIR}/lib/pc_config.h" DESTINATION include)
  install(DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/include" DESTINATION .)

  ### ---[ Configure pgpointcloud-config.cmake
  include(CMakePackageConfigHelpers)

  # Set up our directory structure for output libraries and binaries
  # (Note: these are the build locations, not the install locations)
  if(NOT CMAKE_RUNTIME_OUTPUT_DIRECTORY)
    set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${PGPOINTCLOUD_BINARY_DIR}/bin")
  endif()
  if(NOT CMAKE_LIBRARY_OUTPUT_DIRECTORY)
    if(UNIX)
      set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${PGPOINTCLOUD_BINARY_DIR}/lib")
    else()
      set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${PGPOINTCLOUD_BINARY_DIR}/bin")
    endif()
  endif()
  if(NOT CMAKE_ARCHIVE_OUTPUT_DIRECTORY)
    set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${PGPOINTCLOUD_BINARY_DIR}/lib")
  endif()

  set(version_config "${PROJECT_BINARY_DIR}/pgpointcloud-config-version.cmake")
  set(project_config "${PROJECT_BINARY_DIR}/pgpointcloud-config.cmake")

  # not use install(EXPORT)
  # Generate the version, config and target files into the build directory.
  write_basic_package_version_file(
      ${version_config}
      VERSION ${VERSION}
      COMPATIBILITY AnyNewerVersion)
  configure_package_config_file(
      ${PROJECT_SOURCE_DIR}/pgpointcloud-config.cmake.in
      ${project_config}
      INSTALL_DESTINATION ${PGPOINTCLOUD_CMAKE_DIR})
      # PATH_VARS INCLUDE_INSTALL_DIR SYSCONFIG_INSTALL_DIR)

  install(
      FILES ${project_config} ${version_config}
      DESTINATION ${PGPOINTCLOUD_CMAKE_DIR})

  # Install the export set for use with the install-tree
  install(EXPORT ${targets_export_name} 
          DESTINATION ${PGPOINTCLOUD_CMAKE_DIR}
          FILE ${targets_export_name}.cmake
          NAMESPACE pgpointcloud::)
#endif()

