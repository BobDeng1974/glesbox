#----------------------------------------------------------------------------------------------------------------------
#  Root CMake file for glesbox
#
#  Copyright (c) 2016 xxx
#
#  binfeix.li@intel.com 2015/07/20 v1.0
#
#----------------------------------------------------------------------------------------------------------------------

INCLUDE(build/cmake/version.cmake)
INCLUDE(CheckCXXCompilerFlag)

PROJECT(glesbox)

#----------------------------------------------------------------------------------------------------------------------
#env config
CMAKE_POLICY(SET CMP0014 OLD)
CMAKE_POLICY(SET CMP0000 OLD)
SET(AVATARLIB_VERSION 1.0)

SET(EXECUTABLE_OUTPUT_PATH ${CMAKE_SOURCE_DIR}/bin CACHE PATH "Binary output directory for applications")
#SET(LIBRARY_OUTPUT_PATH ${CMAKE_SOURCE_DIR}/libs CACHE PATH "libraries output directory for applications")
SET(PROJECT_SOURCE_PATH ${CMAKE_SOURCE_DIR}/src CACHE PATH "Source code directory for applications")
SET(PROJECT_BUILD_PATH ${CMAKE_SOURCE_DIR}/build CACHE PATH "build script directory for applications")
SET(PROJECT_INCLUDE_PATH ${CMAKE_SOURCE_DIR}/include CACHE PATH "head-files directory for applications")
SET(PROJECT_SAMPLES_PATH ${CMAKE_SOURCE_DIR}/samples CACHE PATH "sample code directory for applications")
SET(PROJECT_TOOLS_PATH ${CMAKE_SOURCE_DIR}/tools CACHE PATH "tools directory for applications")
SET(PROJECT_DOC_PATH ${CMAKE_SOURCE_DIR}/doc CACHE PATH "doc directory for applications")

#----------------------------------------------------------------------------------------------------------------------
#cross-platform config
IF(WIN32)
  MESSAGE("Compiling on windows")
  ADD_DEFINITIONS(-DWIN32 -D_WIN32)
  IF(${DEBUG})
    MESSAGE("Compling the debug version")
    SET(CMAKE_CXX_FLAGS "/W3 /Od /EHsc")
    SET(CMAKE_C_FLAGS "/W3 /Od /EHsc")
  ELSE()
    MESSAGE("Compling the release version")
    SET(CMAKE_CXX_FLAGS "/W3 /Ox /EHsc")
    SET(CMAKE_C_FLAGS "/W3 /Ox /EHsc")
  ENDIF()
ELSEIF(ANDROID)
  MESSAGE("Compiling on android")
  ENABLE_LANGUAGE(ASM)
  IF(${DEBUG})
    MESSAGE("Compling the debug version")
    SET(CMAKE_CXX_FLAGS "-g -fPIC -fno-inline")
    SET(CMAKE_C_FLAGS "-g -fPIC -fno-inline")
  ELSE()
    MESSAGE("Compling the release version")
    if (ANDROID AND ${ANDROID_ABI} MATCHES "^(x86)+")
      SET(CMAKE_CXX_FLAGS "-O3 -fPIC -mssse3 -fstrict-aliasing -fprefetch-loop-arrays -fno-rtti ${CMAKE_CXX_FLAGS}")
      SET(CMAKE_CXX_FLAGS "-fno-exceptions -finline-functions -funswitch-loops -fgcse-after-reload ${CMAKE_CXX_FLAGS}")
      SET(CMAKE_CXX_FLAGS "-pipe ${CMAKE_CXX_FLAGS}")
      SET(CMAKE_C_FLAGS " -O3 -fPIC -mssse3 -fstrict-aliasing -fprefetch-loop-arrays -finline-functions ${CMAKE_C_FLAGS}")
      SET(CMAKE_C_FLAGS "-funswitch-loops -fgcse-after-reload -pipe ${CMAKE_C_FLAGS}")
    else()
      SET(CMAKE_CXX_FLAGS "-mfpu=neon -O3 -fPIC -fstrict-aliasing -fprefetch-loop-arrays -fno-rtti ${CMAKE_CXX_FLAGS}")
      SET(CMAKE_CXX_FLAGS "-fno-exceptions -finline-functions -funswitch-loops -fgcse-after-reload ${CMAKE_CXX_FLAGS}")
      SET(CMAKE_CXX_FLAGS "-pipe ${CMAKE_CXX_FLAGS}")
      SET(CMAKE_C_FLAGS "-mfpu=neon -O3 -fPIC -fstrict-aliasing -fprefetch-loop-arrays -finline-functions ${CMAKE_C_FLAGS}")
      SET(CMAKE_C_FLAGS "-funswitch-loops -fgcse-after-reload -pipe ${CMAKE_C_FLAGS}")
    endif()
  ENDIF()
ELSEIF(IOS)
  MESSAGE("Compiling on ios")
  ENABLE_LANGUAGE(ASM)
  IF(${DEBUG})
    MESSAGE("Compling the debug version")
    SET(CMAKE_CXX_FLAGS "-g --sysroot ${CMAKE_IOS_SYSROOT} ${CMAKE_CXX_FLAGS}")
    SET(CMAKE_C_FLAGS "-g --sysroot ${CMAKE_IOS_SYSROOT} ${CMAKE_C_FLAGS}")
  ELSE()
    MESSAGE("Compling the release version")
    SET(CMAKE_CXX_FLAGS "-O3 -flto --sysroot ${CMAKE_IOS_SYSROOT} ${CMAKE_CXX_FLAGS}")
    SET(CMAKE_C_FLAGS "-O3 -flto --sysroot ${CMAKE_IOS_SYSROOT} ${CMAKE_C_FLAGS}")
  ENDIF()
ELSEIF(JS)
  MESSAGE("Compiling on emscripten into javascript")
  SET(CMAKE_CXX_FLAGS "-Wno-warn-absolute-paths ${CMAKE_CXX_FLAGS}")
  SET(CMAKE_C_FLAGS "-Wno-warn-absolute-paths ${CMAKE_C_FLAGS}")
  IF(${DEBUG})
    MESSAGE("Compling the debug version")
    SET(CMAKE_CXX_FLAGS "-g3 -fPIC -fno-inline ${CMAKE_CXX_FLAGS}")
    SET(CMAKE_C_FLAGS "-g3 -fPIC -fno-inline ${CMAKE_C_FLAGS}")
  ELSE()
    MESSAGE("Compling the release version")
    SET(CMAKE_CXX_FLAGS "-O3 -flto -fPIC ${CMAKE_CXX_FLAGS}")
    SET(CMAKE_C_FLAGS "-O3 -flto -fPIC ${CMAKE_C_FLAGS}")
  ENDIF()
ELSEIF(UNIX)
  MESSAGE("Compiling on unix-like os")
  IF(${DEBUG})
    MESSAGE("Compling the debug version")
    SET(CMAKE_CXX_FLAGS "-g -fPIC -fno-inline -msse3 -msse2 ${CMAKE_CXX_FLAGS}")
    SET(CMAKE_C_FLAGS "-g -fPIC -fno-inline -msse3 -msse2 ${CMAKE_C_FLAGS}")
  ELSE()
    MESSAGE("Compling the release version")
    SET(CMAKE_CXX_FLAGS "-O3 -flto -fPIC -msse3 -msse2 ${CMAKE_CXX_FLAGS}")
    SET(CMAKE_C_FLAGS "-O3 -flto -fPIC -msse3 -msse2 ${CMAKE_C_FLAGS}")
  ENDIF()
ENDIF()

if (NOT WIN32 AND NOT IOS) #vs2012 support c++11 defaultly
  CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
  CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)
  IF(COMPILER_SUPPORTS_CXX11)
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
  ELSEIF(COMPILER_SUPPORTS_CXX0X)
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
  ELSE()
    MESSAGE(FATAL_ERROR "Compiler ${CMAKE_CXX_COMPILER} has no C++11 support.")
  ENDIF()
ENDIF()

ADD_SUBDIRECTORY(${PROJECT_SOURCE_PATH})
#IF(WIN32)
#  ADD_SUBDIRECTORY(${PROJECT_SAMPLES_PATH})
#ENDIF()
