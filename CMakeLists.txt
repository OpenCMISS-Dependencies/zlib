
 # ***** BEGIN LICENSE BLOCK *****
 # Version: MPL 1.1/GPL 2.0/LGPL 2.1
 #
 # The contents of this file are subject to the Mozilla Public License Version
 # 1.1 (the "License"); you may not use this file except in compliance with
 # the License. You may obtain a copy of the License at
 # http://www.mozilla.org/MPL/
 #
 # Software distributed under the License is distributed on an "AS IS" basis,
 # WITHOUT WARRANTY OF ANY KIND, either express or implied. See the License
 # for the specific language governing rights and limitations under the
 # License.
 #
 # The Original Code is cmgui
 #
 # The Initial Developer of the Original Code is
 # Auckland Uniservices Ltd, Auckland, New Zealand.
 # Portions created by the Initial Developer are Copyright (C) 2005
 # the Initial Developer. All Rights Reserved.
 #
 # Contributor(s): 
 #
 # Alternatively, the contents of this file may be used under the terms of
 # either the GNU General Public License Version 2 or later (the "GPL"), or
 # the GNU Lesser General Public License Version 2.1 or later (the "LGPL"),
 # in which case the provisions of the GPL or the LGPL are applicable instead
 # of those above. If you wish to allow use of your version of this file only
 # under the terms of either the GPL or the LGPL, and not to allow others to
 # use your version of this file under the terms of the MPL, indicate your
 # decision by deleting the provisions above and replace them with the notice
 # and other provisions required by the GPL or the LGPL. If you do not delete
 # the provisions above, a recipient may use your version of this file under
 # the terms of any one of the MPL, the GPL or the LGPL.
 #
 # ***** END LICENSE BLOCK ***** */

CMAKE_MINIMUM_REQUIRED( VERSION 3.0 )
PROJECT(ZLIB VERSION 1.2.3 LANGUAGES C)

option(BUILD_TESTS "${PROJECT_NAME} - Build tests" ON)
option(ZLIB_BUILD_EXAMPLE "Build example application" ${BUILD_TESTS})
option(ZLIB_BUILD_MINIGZIP "Build minigzip application" ${BUILD_TESTS})

# libz is in compliance with ubuntu (the least)
SET(ZLIB_TARGET z)

IF( MSVC )
	ADD_DEFINITIONS( -D_CRT_SECURE_NO_WARNINGS )
ENDIF()

SET( ZLIB_SRCS
	adler32.c
	compress.c
	crc32.c
	deflate.c
	gzio.c
	infback.c
	inffast.c
	inflate.c
	inftrees.c
	trees.c
	uncompr.c
	zutil.c )
SET( ZLIB_HDRS
	crc32.h
	deflate.h
	inffast.h
	inffixed.h
	inflate.h
	inftrees.h
	trees.h
	zutil.h )
SET( ZLIB_PUBLIC_HDRS
	zconf.h
	zlib.h )
SET( EXAMPLE_SRCS
	example.c )
SET( MINIGZIP_SRCS
	minigzip.c )

SET( CMAKE_DEBUG_POSTFIX "d" )
SET( CMAKE_MFC_FLAG 0 )

IF( WIN32 AND BUILD_SHARED_LIBS )
	SET( ZLIB_WIN32_XTRAS 
		win32/zlib.def
		win32/zlib1.rc )
ENDIF()

IF(${CMAKE_SYSTEM_NAME} STREQUAL "Linux" AND ${CMAKE_SYSTEM_PROCESSOR} STREQUAL "x86_64" )
	SET_SOURCE_FILES_PROPERTIES(ZLIB_SRCS
		PROPERTIES COMPILE_FLAGS "-fPIC")
ENDIF()


add_library(${ZLIB_TARGET} ${ZLIB_SRCS}	${ZLIB_HDRS}
	${ZLIB_PUBLIC_HDRS}	${ZLIB_WIN32_XTRAS})

if (BUILD_TESTS)
    if (ZLIB_BUILD_EXAMPLE)
        add_executable( example ${EXCLUDE_EXAMPLE} ${EXAMPLE_SRCS} ${ZLIB_PUBLIC_HDRS} )
        target_link_libraries( example ${ZLIB_TARGET} )
    endif()
    if (ZLIB_BUILD_MINIGZIP)
        add_executable( minigzip ${EXCLUDE_MINIGZIP} ${MINIGZIP_SRCS} ${ZLIB_PUBLIC_HDRS} )
        target_link_libraries( minigzip ${ZLIB_TARGET} )
    endif()
endif()

# Install targets
IF( WIN32 AND BUILD_SHARED_LIBS)
	SET_TARGET_PROPERTIES(${ZLIB_TARGET} PROPERTIES IMPORT_SUFFIX _dll.lib)
	INSTALL(TARGETS ${ZLIB_TARGET} ARCHIVE DESTINATION lib )
ENDIF()

INSTALL(TARGETS ${ZLIB_TARGET}
    EXPORT zlib-config
	DESTINATION lib
	INCLUDES DESTINATION include)
INSTALL(FILES ${ZLIB_PUBLIC_HDRS} DESTINATION include )

install(EXPORT zlib-config DESTINATION lib/cmake)
include(CMakePackageConfigHelpers)
WRITE_BASIC_PACKAGE_VERSION_FILE(${CMAKE_CURRENT_BINARY_DIR}/zlib-config-version.cmake COMPATIBILITY AnyNewerVersion)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/zlib-config-version.cmake DESTINATION lib/cmake)