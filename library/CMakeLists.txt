# don't use this file directly. use the one in the root folder of the project
PROJECT (dfhack-library)
cmake_minimum_required(VERSION 2.6)
SET(CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/CMake/Modules)

# disable warning, autosearch
if(COMMAND cmake_policy)
  cmake_policy(SET CMP0003 NEW)
endif(COMMAND cmake_policy)

if("${CMAKE_SOURCE_DIR}" STREQUAL "${CMAKE_BINARY_DIR}")
   message(SEND_ERROR "In-source builds are not allowed.")
endif("${CMAKE_SOURCE_DIR}" STREQUAL "${CMAKE_BINARY_DIR}")

IF(NOT DEFINED CMAKE_BUILD_TYPE)
  SET(CMAKE_BUILD_TYPE "Release" CACHE STRING "Choose the type of build, options are: None(CMAKE_CXX_FLAGS or CMAKE_C_FLAGS used) Debug Release RelWithDebInfo MinSizeRel.")
ENDIF(NOT DEFINED CMAKE_BUILD_TYPE)

SET( LIBRARY_OUTPUT_PATH ${CMAKE_SOURCE_DIR}/output CACHE PATH "Output directory for the dfhack library" )
SET( EXECUTABLE_OUTPUT_PATH ${CMAKE_SOURCE_DIR}/output CACHE PATH "Output directory for the dfhack tools" )

include_directories (${CMAKE_SOURCE_DIR}/library/include/)
include_directories (${CMAKE_SOURCE_DIR}/library/shm/)
include_directories (${CMAKE_SOURCE_DIR}/library/depends/md5/)
include_directories (${CMAKE_SOURCE_DIR}/library/depends/tinyxml/)
include_directories (${CMAKE_SOURCE_DIR}/library/depends/argstream/)
include_directories (${CMAKE_SOURCE_DIR}/library/private/)

SET(PROJECT_HDRS
private/DFMemInfoManager.h
private/ContextShared.h
private/Internal.h
include/dfhack/DFError.h
include/dfhack/DFMemInfo.h
include/dfhack/DFProcessEnumerator.h
include/dfhack/DFProcess.h
include/dfhack/DFTileTypes.h
include/dfhack/DFTypes.h
include/dfhack/DFVector.h
include/dfhack-c/DFTypes_C.h
include/dfhack-c/DFContext_C.h
include/dfhack/DFIntegers.h
include/dfhack/modules/Buildings.h
include/dfhack/modules/Constructions.h
include/dfhack/modules/Creatures.h
include/dfhack/modules/Gui.h
include/dfhack/modules/Items.h
include/dfhack/modules/Maps.h
include/dfhack/modules/Materials.h
include/dfhack/modules/Position.h
include/dfhack/modules/Translation.h
include/dfhack/modules/Vegetation.h
include/dfhack/modules/WindowIO.h
include/dfhack/modules/World.h
)

SET(PROJECT_SRCS
DFMemInfo.cpp
DFMemInfoManager.cpp
DFContextManager.cpp
DFContext.cpp
DFProcessEnumerator.cpp
ContextShared.cpp
DFContext_C.cpp
DFTypes_C.cpp

depends/md5/md5.cpp
depends/md5/md5wrapper.cpp

depends/tinyxml/tinystr.cpp
depends/tinyxml/tinyxml.cpp
depends/tinyxml/tinyxmlerror.cpp
depends/tinyxml/tinyxmlparser.cpp

modules/Buildings.cpp
modules/Constructions.cpp
modules/Creatures.cpp
modules/Gui.cpp
modules/Items.cpp
modules/Maps.cpp
modules/Materials.cpp
modules/Position.cpp
modules/Translation.cpp
modules/Vegetation.cpp
modules/World.cpp

modules/Buildings_C.cpp
modules/Constructions_C.cpp
modules/Creatures_C.cpp
modules/Gui_C.cpp
modules/Items_C.cpp
modules/Maps_C.cpp
modules/Position_C.cpp
modules/Materials_C.cpp
modules/Translation_C.cpp
modules/Vegetation_C.cpp
modules/WindowIO_C.cpp
modules/World_C.cpp
)

SET(PROJECT_HDRS_LINUX
)

SET(PROJECT_HDRS_WINDOWS
include/dfhack/DFstdint_win.h
)

SET(PROJECT_SRCS_LINUX
DFProcess-linux.cpp
DFProcess-linux-SHM.cpp
DFProcess-linux-wine.cpp
modules/WindowIO-linux.cpp
)

SET(PROJECT_SRCS_WINDOWS
DFProcess-windows.cpp
DFProcess-windows-SHM.cpp
modules/WindowIO-windows.cpp
)

IF(UNIX)
    LIST(APPEND PROJECT_HDRS ${PROJECT_HDRS_LINUX})
    LIST(APPEND PROJECT_SRCS ${PROJECT_SRCS_LINUX})
ELSE(UNIX)
    LIST(APPEND PROJECT_HDRS ${PROJECT_HDRS_WINDOWS})
    LIST(APPEND PROJECT_SRCS ${PROJECT_SRCS_WINDOWS})
ENDIF(UNIX)

SET_SOURCE_FILES_PROPERTIES( ${PROJECT_HDRS} PROPERTIES HEADER_FILE_ONLY TRUE )

LIST(APPEND PROJECT_SRCS ${PROJECT_HDRS})

SET( MEMXML_DATA_PATH . CACHE PATH "search path for Memory.xml")
# OPTION( VARIABLE "Description" Initial state)
#OPTION( WITH_FOO "Enable FOO support" ON )
#OPTION( WITH_BAR "Enable BAR component" OFF )
    
# Are we 64bit? (Damn you, ptrace()!)
IF( CMAKE_SIZEOF_VOID_P MATCHES 4 )
    SET( HAVE_64_BIT 0 )
ELSE( CMAKE_SIZEOF_VOID_P MATCHES 4 )
    SET( HAVE_64_BIT 1 )
ENDIF( CMAKE_SIZEOF_VOID_P MATCHES 4 )

CONFIGURE_FILE( ${CMAKE_SOURCE_DIR}/library/config.h.cmake ${CMAKE_SOURCE_DIR}/library/private/config.h )

ADD_DEFINITIONS(-DBUILD_DFHACK_LIB)

IF(UNIX)
  add_definitions(-DLINUX_BUILD)
  add_definitions(-DUSE_CONFIG_H)
  find_library(X11_LIBRARY X11)
  SET(CMAKE_CXX_FLAGS_DEBUG "-g -Wall")
  SET(CMAKE_CXX_FLAGS "-fvisibility=hidden")
    
  SET(PROJECT_LIBS ${X11_LIBRARY} rt ) #dfhack-md5 dfhack-tixml
ELSE(UNIX)
  SET(PROJECT_LIBS psapi)
ENDIF(UNIX)

ADD_LIBRARY(dfhack SHARED ${PROJECT_SRCS})

SET_TARGET_PROPERTIES(dfhack  PROPERTIES DEBUG_POSTFIX "-debug" )

TARGET_LINK_LIBRARIES(dfhack ${PROJECT_LIBS})

IF(UNIX)
  install(TARGETS dfhack LIBRARY DESTINATION lib)
  install(FILES ${CMAKE_SOURCE_DIR}/output/Memory.xml DESTINATION share/dfhack)
ENDIF(UNIX)
