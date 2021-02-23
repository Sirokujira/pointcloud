#------------------------------------------------------------------------------
# generate config include

configure_file (
  "${PROJECT_SOURCE_DIR}/pgsql_postgis/pointcloud_postgis.sql.cmake"
  "${PROJECT_SOURCE_DIR}/pgsql_postgis/pointcloud_postgis.sql"
  )

configure_file (
  "${PROJECT_SOURCE_DIR}/pgsql_postgis/pointcloud_postgis.control.cmake"
  "${PROJECT_SOURCE_DIR}/pgsql_postgis/pointcloud_postgis.control"
  )

set ( PCPG_INSTALL_EXENSIONS
  pointcloud_postgis.sql
  pointcloud_postgis.control
  )
        
install (
  FILES ${PCPG_INSTALL_EXENSIONS} 
  DESTINATION "${PGSQL_SHAREDIR}/extension"
  )


