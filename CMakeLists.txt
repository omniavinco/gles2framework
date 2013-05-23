cmake_minimum_required(VERSION 2.6)
project( gles2framework )

SET( CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/lib )
SET( CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/lib )
SET( CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/bin )

if(MINGW OR UNIX)
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wno-attributes -std=gnu99")
  set_property(GLOBAL PROPERTY COMPILE_FLAGS "/ZI")
endif()

if(BUILD_RPI OR BUILD_RPI_NO_X OR BUILD_GLFW)
else()
  if(CMAKE_SYSTEM_NAME)
    if(CMAKE_SYSTEM_NAME STREQUAL "rpi")
      set(BUILD_RPI ON)
    elseif(CMAKE_SYSTEM_NAME STREQUAL "rpi_noX")
      set(BUILD_RPI_NO_X ON)
    else()
      set(BUILD_GLFW ON)
    endif()
  else()
    set(BUILD_GLFW ON)
  endif()
endif()

include_directories( include )
include_directories( kazmath/kazmath )

if(BUILD_RPI OR BUILD_RPI_NO_X)
	if(NOT RPI_FIRMWARE_ROOT)
    SET(RPI_FIRMWARE_ROOT "")
  endif()
  include_directories("${RPI_FIRMWARE_ROOT}/opt/vc/include" "${RPI_FIRMWARE_ROOT}/opt/vc/include/interface/vcos/pthreads" "${RPI_FIRMWARE_ROOT}/opt/vc/include/interface/vmcs_host" "${RPI_FIRMWARE_ROOT}/opt/vc/include/interface/vmcs_host/linux")
  link_directories("${RPI_FIRMWARE_ROOT}/opt/vc/lib")
endif()

if(BUILD_RPI)
  add_definitions( "-D__FOR_RPi__")
elseif(BUILD_RPI_NO_X)
  add_definitions( "-D__FOR_RPi_noX__")
elseif(BUILD_GLFW)
  if( WIN32 )
    include_directories( glew/include )
    link_directories( ${CMAKE_CURRENT_SOURCE_DIR}/glew/lib )
  endif( WIN32 )
  include_directories( glfw/include )

  link_directories( ${CMAKE_CURRENT_SOURCE_DIR}/glfw/lib )

  add_definitions( "-D__FOR_GLFW__" )
endif()

file( GLOB GLES2FRAMEWORK_HEADERS include/*.h )
file( GLOB GLES2FRAMEWORK_SOURCES src/*.c )
file( GLOB_RECURSE KAZMATH_HEADERS kazmath/kazmath/*.h )
file( GLOB_RECURSE KAZMATH_SOURCES kazmath/kazmath/*.c )

source_group( kazmath FILES ${KAZMATH_HEADERS} ${KAZMATH_SOURCES})
source_group( inc FILES ${GLES2FRAMEWORK_HEADERS} )
source_group( src FILES ${GLES2FRAMEWORK_SOURCES} )

add_library( gles2framework STATIC 
	${GLES2FRAMEWORK_HEADERS} 
	${GLES2FRAMEWORK_SOURCES} 
	${KAZMATH_HEADERS}
	${KAZMATH_SOURCES} )

add_library( gles2framework_shared SHARED
	${GLES2FRAMEWORK_HEADERS}
	${GLES2FRAMEWORK_SOURCES}
	${KAZMATH_HEADERS}
	${KAZMATH_SOURCES} )
set_property( TARGET gles2framework_shared PROPERTY LIBRARY_OUTPUT_NAME gles2framework )
	
file( GLOB EXAMPLE_SIMPLE examples/simple.c )
file( GLOB EXAMPLE_SPRITES examples/sprites.c )
file( GLOB EXAMPLE_INVADERS examples/invaders.c )
#file( GLOB EXAMPLE_PHYSTEST examples/phystest.c )
#file( GLOB EXAMPLE_CHIPTEST examples/chiptest.c )

file(GLOB C_FILES 
	${GLES2FRAMEWORK_SOURCES}
	${KAZMATH_SOURCES}
	${EXAMPLE_SIMPLE}
	${EXAMPLE_SPRITES}
	${EXAMPLE_INVADERS}
	#${EXAMPLE_PHYSTEST}
#	${EXAMPLE_CHIPTEST}
	)

if( BUILD_GLFW )	
  set_source_files_properties(${C_FILES} PROPERTIES LANGUAGE CXX)
endif()

# Executables for only Standalone project
if(${CMAKE_CURRENT_SOURCE_DIR} STREQUAL ${CMAKE_SOURCE_DIR})

  add_executable( simple ${EXAMPLE_SIMPLE} )
  add_executable( sprites ${EXAMPLE_SPRITES} )
  add_executable( invaders ${EXAMPLE_INVADERS} )
  #add_executable( phystest ${EXAMPLE_PHYSTEST} )
  #add_executable( chiptest ${EXAMPLE_CHIPTEST} )

  if(BUILD_RPI OR BUILD_RPI_NO_X)
    LIST( APPEND SAMPLE_LINK_LIBRARIES gles2framework GLESv2 EGL bcm_host m)
		if(BUILD_RPI)
			LIST( APPEND SAMPLE_LINK_LIBRARIES X11 )
		endif()
  elseif(BUILD_GLFW)
    LIST( APPEND SAMPLE_LINK_LIBRARIES gles2framework GLFW )
    if( WIN32 )
      LIST( APPEND SAMPLE_LINK_LIBRARIES glew32 opengl32 )
    elseif( APPLE )
      LIST( APPEND SAMPLE_LINK_LIBRARIES GL )
    endif( WIN32 )
  endif()

  target_link_libraries( simple ${SAMPLE_LINK_LIBRARIES} )
  target_link_libraries( sprites ${SAMPLE_LINK_LIBRARIES} )
  target_link_libraries( invaders ${SAMPLE_LINK_LIBRARIES} )
  #target_link_libraries( phystest ${SAMPLE_LINK_LIBRARIES} )
  #target_link_libraries( chiptest ${SAMPLE_LINK_LIBRARIES} )

endif()
