cmake_minimum_required(VERSION 2.8.6)
project(subhook C)

if(POLICY CMP0042)
  cmake_policy(SET CMP0042 NEW)
endif()

list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

set(SUBHOOK_VERSION_MAJOR 0)
set(SUBHOOK_VERSION_MINOR 4)
set(SUBHOOK_VERSION_PATCH 2)

set(SUBHOOK_VERSION ${SUBHOOK_VERSION_MAJOR})
set(SUBHOOK_VERSION ${SUBHOOK_VERSION}.${SUBHOOK_VERSION_MINOR})
set(SUBHOOK_VERSION ${SUBHOOK_VERSION}.${SUBHOOK_VERSION_PATCH})

set(SUBHOOK_STATIC OFF CACHE BOOL "Build as a static library")
set(SUBHOOK_INSTALL ON CACHE BOOL
  "Enable installation and packaging of targets/files with CPack")
set(SUBHOOK_TESTS ON CACHE BOOL "Enable tests")
set(SUBHOOK_FORCE_32BIT OFF CACHE BOOL
  "Configure for compiling 32-bit binaries (on 64-bit systems)")

set(SUBHOOK_HEADERS subhook.h)
set(SUBHOOK_SOURCES subhook.c subhook_private.h)

add_definitions(-DSUBHOOK_IMPLEMENTATION)

if(SUBHOOK_STATIC)
  add_library(subhook STATIC ${SUBHOOK_HEADERS} ${SUBHOOK_SOURCES})
else()
  add_library(subhook SHARED ${SUBHOOK_HEADERS} ${SUBHOOK_SOURCES})
endif()

include_directories(${CMAKE_CURRENT_SOURCE_DIR})
set_property(DIRECTORY ${CMAKE_SOURCE_DIR}
             APPEND PROPERTY INCLUDE_DIRECTORIES ${CMAKE_CURRENT_SOURCE_DIR})

if(SUBHOOK_FORCE_32BIT)
  if(APPLE)
    set_target_properties(subhook PROPERTIES OSX_ARCHITECTURES i386)
  endif()
  if(UNIX)
    set_property(TARGET subhook APPEND_STRING PROPERTY
                 COMPILE_FLAGS " -m32")
    set_property(TARGET subhook APPEND_STRING PROPERTY LINK_FLAGS " -m32")
  endif()
endif()

if(SUBHOOK_STATIC)
  add_definitions(-DSUBHOOK_STATIC)
  set_property(DIRECTORY ${CMAKE_SOURCE_DIR}
               APPEND PROPERTY COMPILE_DEFINITIONS SUBHOOK_STATIC)
endif()

set_property(GLOBAL PROPERTY USE_FOLDERS ON)

if(SUBHOOK_INSTALL)
  install(TARGETS subhook LIBRARY DESTINATION lib
                          ARCHIVE DESTINATION lib
                          RUNTIME DESTINATION bin)
  install(FILES ${SUBHOOK_HEADERS} DESTINATION include)
endif()

set(CPACK_PACKAGE_NAME ${PROJECT_NAME})
set(CPACK_PACKAGE_VERSION_MAJOR ${SUBHOOK_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${SUBHOOK_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH ${SUBHOOK_VERSION_PATCH})

include(CPack)
include(CTest)

if(BUILD_TESTING AND SUBHOOK_TESTS)
  enable_testing()
  add_subdirectory(test)
endif()
