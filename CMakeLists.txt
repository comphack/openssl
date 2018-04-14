# Based on original work by David Manura
# Copyright (C) 2007-2012 LuaDist.
# Copyright (C) 2013 Brian Sidebotham

# Redistribution and use of this file is allowed according to the terms of the
# MIT license.
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in all
# copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
# SOFTWARE.

cmake_minimum_required( VERSION 2.8.3 )

project( openssl )

option(USE_STATIC_RUNTIME "Use the static MSVC runtime." OFF)

if( "${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC" AND USE_STATIC_RUNTIME )
  set( CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} /MTd" )
  set( CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /MTd" )

  set( CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} /MT" )
  set( CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /MT" )
elseif( "${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC" )
  set( CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} /MDd" )
  set( CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /MDd" )

  set( CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} /MD" )
  set( CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /MD" )
else()
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fPIC")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC")
endif()

include_directories(
  ${CMAKE_SOURCE_DIR}
  ${CMAKE_BINARY_DIR}
  ${CMAKE_BINARY_DIR}/openssl
)

set(BUILD_OBJECT_LIBRARY_ONLY "OFF" CACHE BOOL "Build object library only")

option(BUILD_VALGRIND_FRIENDLY "Build with -DPURIFY so Valgrind will not show errors in OpenSSL's PRNG." OFF)

if( BUILD_VALGRIND_FRIENDLY )
  add_definitions( -DPURIFY )
endif()

add_definitions( -DOPENSSL_NO_ASM )
add_definitions( -DOPENSSL_THREADS )

if( WIN32 AND NOT CYGWIN )
  add_definitions( -DOPENSSL_SYSNAME_WIN32 )
  add_definitions( -DWIN32_LEAN_AND_MEAN )
endif ( )

if( MINGW )
  set( CMAKE_SHARED_LINKER_FLAGS "-Wl,--export-all" )
endif()

add_subdirectory( crypto )
add_subdirectory( ssl )

file( COPY e_os2.h DESTINATION ${CMAKE_BINARY_DIR}/openssl )

if(${BUILD_OBJECT_LIBRARY_ONLY})
  file( COPY e_os2.h DESTINATION ${CMAKE_CURRENT_BINARY_DIR} )
else()
  #add_subdirectory( apps )

  install( FILES e_os2.h DESTINATION include/openssl )

  # Generate the package target
  set( CPACK_GENERATOR ZIP TGZ )
  set( CPACK_PACKAGE_NAME "openssl-cmake" )
  set( CPACK_PACKAGE_VERSION_MAJOR 1  )
  set( CPACK_PACKAGE_VERSION_MINOR 0  )
  set( CPACK_PACKAGE_VERSION_PATCH 1e )

  include( CPack )
endif()

# Include the PDB file if on Windows
if(WIN32)
    install(FILES $<TARGET_PBB_FILE:ssleay32>
      DESTINATION lib)
    install(FILES $<TARGET_PDB_FILE:libeay32>
      DESTINATION lib)
endif(WIN32)
