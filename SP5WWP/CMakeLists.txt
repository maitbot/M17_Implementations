# SPDX-License-Identifier: GPL-2.0-or-later
cmake_minimum_required(VERSION 3.17)
project(libm17 LANGUAGES C VERSION 0.1
  HOMEPAGE_URL "https://m17project.org/"
  DESCRIPTION "M17_Implementations from SP5WWP.")
  set(CMAKE_CXX_STANDARD 11)
  set(CMAKE_CXX_STANDARD_REQUIRED TRUE)
  set(CMAKE_VERBOSE_MAKEFILE ON)
  set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
  include(GNUInstallDirs)
  include(CMakeDependentOption)
  include(CTest)
  enable_testing()

option(BUILD_SHARED_LIB "Build shared library" ON)
option(BUILD_STATIC_LIB "Build static library" ON)
# Check that we're building at least one library
if( NOT ${BUILD_SHARED_LIB} AND NOT ${BUILD_STATIC_LIB} )
  message( FATAL_ERROR "Not building static or shared library - at least one must be built")
endif()

cmake_dependent_option(BUILD_STATIC_BINS "Build using static library" OFF "BUILD_STATIC_LIB" OFF)
if( BUILD_STATIC_BINS )
  set(libm17 m17-static)
else()
    set(libm17 m17-shared)
endif()

if(MSVC)
   add_compile_options (/W4 /WX)
else()
   add_compile_options(-Wall -Wextra -pedantic)
endif()

if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
   add_compile_options (-Wformat -Wformat-security -Werror=format-security)
   add_compile_options (-DFORTIFY_SOURCE=2 -fstack-protector-strong -fPIC -pie)
   add_compile_options (-Wl,-z,relro -Wl,-z,now)
   add_compile_options (-O3 -mavx2)
endif()

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE "RelWithDebInfo")
  message(STATUS "Build type not specified: defaulting to RelWithDebInfo.")
endif()

set(libm17_SOURCES
lib/encode/convol.c
lib/encode/symbols.c
lib/decode/viterbi.c
lib/decode/symbols.c
lib/phy/sync.c
lib/phy/interleave.c
lib/phy/randomize.c
lib/math/rrc.c
lib/math/math.c
lib/math/golay.c
lib/lib.c
lib/payload/call.c
lib/payload/crc.c)

if ( BUILD_STATIC_LIB )
 add_library(m17-static STATIC ${libm17_SOURCES})
 target_include_directories(m17-static PUBLIC lib
  $<INSTALL_INTERFACE:include>
  $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/include>
 )
 set_target_properties(m17-static PROPERTIES OUTPUT_NAME m17)
 target_link_libraries(m17-static PUBLIC -lm)
 install(TARGETS m17-static)
endif()

if (BUILD_SHARED_LIB )
 add_library(m17-shared SHARED ${libm17_SOURCES})
 target_include_directories(m17-shared PUBLIC lib
  $<INSTALL_INTERFACE:include>
  $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/include>
 )
 set_target_properties(m17-shared PROPERTIES
    MACOSX_RPATH TRUE
    OUTPUT_NAME m17
    VERSION "${PROJECT_VERSION}"
    SOVERSION "${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}")
 target_link_libraries(m17-shared PUBLIC -lm)
 install(TARGETS m17-shared)
endif()

add_executable(m17-coder-sym m17-coder/m17-coder-sym.c)
target_link_libraries(m17-coder-sym ${libm17})

add_executable(m17-decoder-sym m17-decoder/m17-decoder-sym.c)
target_link_libraries(m17-decoder-sym ${libm17})

add_executable(m17-packet-encode m17-packet/m17-packet-encode.c)
target_link_libraries(m17-packet-encode ${libm17})

add_executable(m17-packet-decode m17-packet/m17-packet-decode.c)
target_link_libraries(m17-packet-decode ${libm17})

install(TARGETS m17-coder-sym m17-decoder-sym m17-packet-encode m17-packet-decode)

if(EXISTS "include/m17/m17.h")
  install(FILES include/m17/m17.h DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/m17/)
else()

install(FILES
lib/encode/convol.h
lib/encode/symbols.h
DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/m17/encode/)

install(FILES
lib/decode/viterbi.h
lib/decode/symbols.h
DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/m17/decode/)

install(FILES
lib/phy/sync.h
lib/phy/randomize.h
lib/phy/interleave.h
DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/m17/phy/)

install(FILES
lib/math/golay.h
lib/math/rrc.h
lib/math/math.h
DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/m17/math/)

install(FILES
lib/payload/crc.h
lib/payload/call.h
lib/payload/lsf.h
DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/m17/payload/)

install(FILES
lib/lib.h
DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/m17/)

endif()

add_test(NAME m17-packet-encode_runs
 COMMAND sh -c "echo -en \"\\x05Testing M17 packet mode.\" | ./m17-packet-encode -S N0CALL -D ALL -C 0 -n 25 -o baseband.rrc")
