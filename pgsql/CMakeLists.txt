

set ( PC_SOURCES
  pc_access.c 
  pc_inout.c      
  pc_pgsql.c       
  )

set ( PC_HEADERS
  pc_pgsql.h
  )

#------------------------------------------------------------------------------
# generate config include

configure_file (
  "${PROJECT_SOURCE_DIR}/pgsql/pointcloud.sql.cmake"
  "${PROJECT_SOURCE_DIR}/pgsql/pointcloud.sql"
  )

configure_file (
  "${PROJECT_SOURCE_DIR}/pgsql/pointcloud.control.cmake"
  "${PROJECT_SOURCE_DIR}/pgsql/pointcloud.control"
  )

set ( PC_INSTALL_EXENSIONS
  pointcloud.sql
  pointcloud.control
  )
        

#include_directories ("${PGSQL_INCLUDEDIR_SERVER}")
#PostgreSQL_TYPE_INCLUDE_DIR
include_directories ("${PROJECT_SOURCE_DIR}/lib")
if (WIN32)
  include_directories ("${PGSQL_INCLUDEDIR_SERVER}/port/win32")
endif (WIN32)

add_library (pointcloud MODULE ${PC_SOURCES} ${PC_HEADERS})

if (APPLE)
  set (PC_LINK_FLAGS "-bundle_loader ${PGSQL_BINDIR}/postgres")
else (APPLE)
  set (PC_LINK_FLAGS " ")
endif (APPLE)

set_target_properties (pointcloud
  PROPERTIES
  OUTPUT_NAME "pointcloud"
  LINK_FLAGS ${PC_LINK_FLAGS}
  PREFIX ""
  )

target_link_libraries (pointcloud libpc-static)
target_link_libraries (pointcloud xml2)
target_link_libraries (pointcloud z)
if (LIBGHT_FOUND)
  target_link_libraries (pointcloud ght)
endif (LIBGHT_FOUND)


#install (
#  TARGETS pointcloud 
#  DESTINATION ${PGSQL_PKGLIBDIR}
#  )
  
# nstall (
#  FILES ${PC_INSTALL_EXENSIONS} 
#  DESTINATION "${PGSQL_SHAREDIR}/extension"
#  )

install(TARGETS pointcloud EXPORT ${targets_export_name}
            RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
            ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
            LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR})

