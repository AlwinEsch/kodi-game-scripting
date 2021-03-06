cmake_minimum_required(VERSION 3.5)
project(mygame)

list(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR})

include(ExternalProject)

string(TOUPPER ${PROJECT_NAME} PROJECT_NAME_UC)

set(LIBRETRO_BINARY_DIR )
set(LIBRETRO_SONAME ${CMAKE_SHARED_LIBRARY_SUFFIX})
set(LIBRETRO_JNISONAME ${CMAKE_SHARED_LIBRARY_SUFFIX})

if(CMAKE_BUILD_TYPE MATCHES Debug)
  set(LIBRETRO_DEBUG DEBUG=1)
endif()

if(CORE_SYSTEM_NAME STREQUAL windows)
  find_package(MinGW REQUIRED)
  set(BUILD_COMMAND ${MINGW_MAKE} -j$ENV{NUMBER_OF_PROCESSORS} -C  -f  platform=win  ${LIBRETRO_DEBUG} GIT_VERSION=)
elseif(CORE_SYSTEM_NAME STREQUAL linux)
  set(BUILD_COMMAND $(MAKE) -C  -f   platform=unix ${LIBRETRO_DEBUG} GIT_VERSION=)
elseif(CORE_SYSTEM_NAME STREQUAL osx)
  set(BUILD_COMMAND $(MAKE) -C  -f   platform=osx ${LIBRETRO_DEBUG} GIT_VERSION=)
elseif(CORE_SYSTEM_NAME STREQUAL ios OR CORE_SYSTEM_NAME STREQUAL darwin_embedded)
  set(LIBRETRO_SONAME _ios${CMAKE_SHARED_LIBRARY_SUFFIX})
  set(BUILD_COMMAND IOSSDK=${CMAKE_OSX_SYSROOT} $(MAKE) -C  -f   platform=ios ${LIBRETRO_DEBUG} GIT_VERSION=)
elseif(CORE_SYSTEM_NAME STREQUAL android)
  if(NOT NDKROOT)
    message(FATAL_ERROR "${PROJECT_NAME} needs NDKROOT for Android. Missing Toolchain file?")
  endif()
  if(CPU STREQUAL armeabi-v7a)
    set(PLATFORM android-armv7)
  elseif(CPU STREQUAL i686)
    set(PLATFORM x86)
  else()
    message(FATAL_ERROR "${PROJECT_NAME} needs Android ${CPU} build command in CMakeLists.txt!")
  endif()
  set(LIBRETRO_SONAME _android${CMAKE_SHARED_LIBRARY_SUFFIX})
  get_filename_component(TOOLCHAIN_DIR ${CMAKE_C_COMPILER} DIRECTORY)
  set(BUILD_COMMAND PATH=${TOOLCHAIN_DIR}:$ENV{PATH} $(MAKE) -C  -f   platform=${PLATFORM} ${LIBRETRO_DEBUG} GIT_VERSION=)
elseif(CORE_SYSTEM_NAME STREQUAL rbpi)
  message(FATAL_ERROR "${PROJECT_NAME} needs RPi build command in CMakeLists.txt!")
elseif(CORE_SYSTEM_NAME STREQUAL freebsd)
  set(BUILD_COMMAND $(MAKE) -C  -f   platform=unix ${LIBRETRO_DEBUG} GIT_VERSION=)
else()
  message(FATAL_ERROR "${PROJECT_NAME} - Unknown system: ${CORE_SYSTEM_NAME}")
endif()

externalproject_add(${PROJECT_NAME}
                    SOURCE_DIR ${PROJECT_SOURCE_DIR}
                    PREFIX ${PROJECT_NAME}
                    CONFIGURE_COMMAND ""
                    INSTALL_COMMAND ""
                    BUILD_COMMAND ${BUILD_COMMAND}
                    BUILD_IN_SOURCE 1)

# install the generated shared library
install(FILES ${PROJECT_SOURCE_DIR}/${LIBRETRO_BINARY_DIR}/${LIBRETRO_SONAME}
        DESTINATION ${CMAKE_INSTALL_PREFIX}/lib/libretro RENAME ${CMAKE_SHARED_LIBRARY_SUFFIX})
