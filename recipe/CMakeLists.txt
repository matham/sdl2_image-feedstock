# Written by Daniel Schreij (dschreij@gmail.com) on 1 Nov 2016
# This procedure is configured to work in an Anaconda build environment (contiuum.io)
# In this environment, it should be called like 
# 
#    %LIBRARY_BIN%\cmake -G "NMake Makefiles" -DCMAKE_INSTALL_PREFIX:PATH="%LIBRARY_PREFIX%" -DCMAKE_BUILD_TYPE:STRING=Release ..
# 
# to be correctly configured
# 
# This file is intended to be used for Windows only! For other platforms, the supplied makefiles work very well.

set(PROJECT_NAME "SDL2_image")
project(${PROJECT_NAME})
CMAKE_MINIMUM_REQUIRED(VERSION 3.0.0)

set(IS_LIBRARY true)
set(CMAKE_INCLUDE_CURRENT_DIR TRUE)

# General source files
file(GLOB SDL2_IMAGE_SOURCE_FILES  ${CMAKE_CURRENT_SOURCE_DIR}/*.c)
file(GLOB SDL2_IMAGE_INCLUDE_FILES ${CMAKE_CURRENT_SOURCE_DIR}/*.h)

include(GenerateExportHeader)

## Try to find SDL2 libraries
# SDL.h
FIND_PATH(SDL2_INCLUDE_DIR
    SDL.h
    PATH_SUFFIXES include/SDL2 include
)

# SDL2.lib
FIND_LIBRARY(SDL2_LIBRARIES
    NAMES SDL2
    PATH_SUFFIXES lib64 lib lib/x64 lib/x86
)

if(SDL2_LIBRARIES)
  message(STATUS "Found SDL2: ${SDL2_LIBRARIES}")
else()
  message(FATAL_ERROR "SDL2 libraries could not be located." )
endif()

# Create target library specs
include_directories(${SDL2_INCLUDE_DIR})
add_library(${PROJECT_NAME} SHARED ${SDL2_IMAGE_SOURCE_FILES})
set_target_properties(${PROJECT_NAME} PROPERTIES DEFINE_SYMBOL "DLL_EXPORT")
set_target_properties(${PROJECT_NAME} PROPERTIES LINK_FLAGS_RELEASE "/NODEFAULTLIB:libcmt.lib")
target_link_libraries(${PROJECT_NAME} ${SDL2_LIBRARIES})

set(SDLIMAGE_SUPPORT_BMP ON CACHE BOOL "Support for BMP file format")
set(SDLIMAGE_SUPPORT_GIF ON CACHE BOOL "Support for GIF file format")
set(SDLIMAGE_SUPPORT_LBM ON CACHE BOOL "Support for LBM file format")
set(SDLIMAGE_SUPPORT_PCX ON CACHE BOOL "Support for PCX file format")
set(SDLIMAGE_SUPPORT_PNM ON CACHE BOOL "Support for PNM file format")
set(SDLIMAGE_SUPPORT_TGA ON CACHE BOOL "Support for TGA file format")
set(SDLIMAGE_SUPPORT_XPM ON CACHE BOOL "Support for XPM file format")
set(SDLIMAGE_SUPPORT_XV ON CACHE BOOL "Support for XV file format")

set(SDLIMAGE_SUPPORT_JPEG ON CACHE BOOL "Support for JPEG file format")
set(SDLIMAGE_SUPPORT_PNG ON CACHE BOOL "Support for PNG file format")
set(SDLIMAGE_SUPPORT_WEBP ON CACHE BOOL "Support for WEBP file format")
set(SDLIMAGE_SUPPORT_TIFF ON CACHE BOOL "Support for TIFF file format")

if(SDLIMAGE_SUPPORT_BMP)
  add_definitions(-DLOAD_BMP)
endif()
if(SDLIMAGE_SUPPORT_GIF)
  add_definitions(-DLOAD_GIF)
endif()
if(SDLIMAGE_SUPPORT_LBM)
  add_definitions(-DLOAD_LBM)
endif()
if(SDLIMAGE_SUPPORT_PCX)
  add_definitions(-DLOAD_PCX)
endif()
if(SDLIMAGE_SUPPORT_PNM)
  add_definitions(-DLOAD_PNM)
endif()
if(SDLIMAGE_SUPPORT_TGA)
  add_definitions(-DLOAD_TGA)
endif()
if(SDLIMAGE_SUPPORT_XPM)
  add_definitions(-DLOAD_XPM)
endif()
if(SDLIMAGE_SUPPORT_XV)
  add_definitions(-DLOAD_XV)
endif()

if(SDLIMAGE_SUPPORT_JPEG)
  include(FindJPEG)
  if(${JPEG_FOUND})
    include_directories(${JPEG_INCLUDE_DIRS})
    target_link_libraries(${PROJECT_NAME} ${JPEG_LIBRARIES})
    add_definitions(-DLOAD_JPG)
  else()
    message(ERROR "Could not find JPEG library. Compiling without.")
  endif() 
endif()

if(SDLIMAGE_SUPPORT_PNG)
  include(FindPNG)
  if(${PNG_FOUND})
    set(PNG_SHARED ON)
    include_directories(${PNG_INCLUDE_DIRS})
    target_link_libraries(${PROJECT_NAME} ${PNG_LIBRARIES})
    add_definitions(${PNG_DEFINITIONS})
    add_definitions(-DPNG_USE_DLL)
    add_definitions(-DLOAD_PNG)
    add_definitions(-DLOAD_PNG_DYNAMIC="libpng16.dll")

    include(FindZLIB)
    if(${ZLIB_FOUND})
      add_definitions(-DZLIB_DLL)
      include_directories(${ZLIB_INCLUDE_DIRS})
      target_link_libraries(${PROJECT_NAME} ${ZLIB_LIBRARIES})
    else()
      message(WARNING "libpng requires zlib, which could not be found.")
    endif()
  else()
    message(ERROR "Could not find PNG library. Compiling without.")
  endif() 
endif()

if(SDLIMAGE_SUPPORT_TIFF)
  include(FindTIFF)
  if(${TIFF_FOUND})
    target_link_libraries(${PROJECT_NAME} ${TIFF_LIBRARIES})
    include_directories(${TIFF_INCLUDE_DIRS})
    add_definitions(-DLOAD_TIF)
    add_definitions(-DLOAD_TIF_DYNAMIC="libtiff.dll")
  else()
    message(ERROR "Could not find TIFF library. Compiling without.")
  endif()
endif()

if(SDLIMAGE_SUPPORT_WEBP)
  # headers
  FIND_PATH(LIBWEBP_INCLUDE_DIR
    decode.h
    PATH_SUFFIXES include/webp include
  )

  # libwebp.lib
  FIND_LIBRARY(LIBWEBP_LIBRARIES
      NAMES libwebp
      PATH_SUFFIXES lib
  )
    
  if(NOT LIBWEBP_LIBRARIES-NOTFOUND)
    target_link_libraries(${PROJECT_NAME} ${LIBWEBP_LIBRARIES})
    include_directories(${LIBWEBP_INCLUDE_DIR})
    add_definitions(-DLOAD_WEBP)
  else()
    message(FATAL_ERROR "Could not locate libwebp.lib")
  endif() 
endif()

##### Installation targets #####
install(TARGETS ${PROJECT_NAME}
  LIBRARY DESTINATION "lib${LIB_SUFFIX}"
  ARCHIVE DESTINATION "lib${LIB_SUFFIX}"
  RUNTIME DESTINATION bin)

# Copy .h files to include dir.
install(FILES ${SDL2_IMAGE_INCLUDE_FILES} DESTINATION include/SDL2)

