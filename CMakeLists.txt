########################################################
# cmake file for building LArReco
# @author Jan Engels, DESY
CMAKE_MINIMUM_REQUIRED(VERSION 2.8.2 FATAL_ERROR)
########################################################


# project name
PROJECT( LArReco )


# project version
SET( LArReco_VERSION_MAJOR 0 )
SET( LArReco_VERSION_MINOR 6 )
SET( LArReco_VERSION_PATCH 0 )


### DEPENDENCIES ############################################################
INCLUDE( Default_settings )

FIND_PACKAGE( LArContent 0.17 REQUIRED )
FIND_PACKAGE( PandoraSDK 0.17 REQUIRED )

FOREACH( pkg LArContent PandoraSDK )
    IF( ${pkg}_FOUND )
        INCLUDE_DIRECTORIES( ${${pkg}_INCLUDE_DIRS} )
        LINK_LIBRARIES( ${${pkg}_LIBRARIES} )
        ADD_DEFINITIONS ( ${${pkg}_DEFINITIONS} )
    ENDIF()
ENDFOREACH()

IF( PANDORA_MONITORING )
    LIST( APPEND CMAKE_MODULE_PATH "$ENV{ROOTSYS}/etc/cmake/" )
    ADD_DEFINITIONS( "-DMONITORING" )
    FIND_PACKAGE( PandoraMonitoring 0.17 REQUIRED )
    FIND_PACKAGE( ROOT 5.26.00 REQUIRED COMPONENTS Eve Geom RGL EG ) # minimum required ROOT version

    FOREACH( pkg PandoraMonitoring ROOT )
        IF( ${pkg}_FOUND )
            INCLUDE_DIRECTORIES( ${${pkg}_INCLUDE_DIRS} )
            LINK_LIBRARIES( ${${pkg}_LIBRARIES} )
            ADD_DEFINITIONS ( ${${pkg}_DEFINITIONS} )
        ENDIF()
    ENDFOREACH()
ENDIF()


### DOCUMENTATION ############################################################
OPTION( INSTALL_DOC "Set to OFF to skip build/install Documentation" OFF )

IF( INSTALL_DOC AND EXISTS "${PROJECT_SOURCE_DIR}/doc/CMakeLists.txt" )
    ADD_SUBDIRECTORY( ./doc )
ELSE()
    MESSAGE( STATUS "INSTALL_DOC set to OFF" )
    SET( INSTALL_DOC OFF )
ENDIF()


### SOURCE ##################################################################

#include directories
INCLUDE_DIRECTORIES( ./include )

# require proper C++
ADD_DEFINITIONS( "-Wall -ansi -pedantic -std=c++0x" )

# need long long for int64 for now
ADD_DEFINITIONS( "-Wno-long-long -Wno-sign-compare -fno-strict-aliasing" )

AUX_SOURCE_DIRECTORY( ./src sources )
ADD_SHARED_LIBRARY( ${PROJECT_NAME} ${sources} )
INSTALL_SHARED_LIBRARY( ${PROJECT_NAME} DESTINATION lib )

# make standalone executables
SET( PROGRAMS1 PandoraInterface )

FOREACH( program ${PROGRAMS1} )
    ADD_EXECUTABLE( ${program} ${PROJECT_SOURCE_DIR}/test/${program}.cxx )
    TARGET_LINK_LIBRARIES( ${program} ${PROJECT_NAME} )

    # install executable
    INSTALL( TARGETS ${program} DESTINATION bin PERMISSIONS
        OWNER_READ OWNER_WRITE OWNER_EXECUTE
        GROUP_READ GROUP_EXECUTE
        WORLD_READ WORLD_EXECUTE )
ENDFOREACH( program )

# display some variables and write them to cache
DISPLAY_STD_VARIABLES()