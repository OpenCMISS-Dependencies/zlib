
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

# libz is in compliance with ubuntu (the least)
SET(LIBRARY_TARGET_NAME z)
STRING( TOUPPER ${LIBRARY_TARGET_NAME} UPPERCASE_LIBRARY_TARGET_NAME )

MACRO( OPTION_WITH_DEFAULT OPTION_NAME OPTION_STRING OPTION_DEFAULT )
	IF( NOT DEFINED ${OPTION_NAME} )
		SET( ${OPTION_NAME} ${OPTION_DEFAULT} )
	ENDIF()
	OPTION( ${OPTION_NAME} "${OPTION_STRING}" ${${OPTION_NAME}} )
ENDMACRO()

MACRO(CACHE_VAR_WITH_DEFAULT OPTION_NAME OPTION_DEFAULT OPTION_TYPE OPTION_STRING )
	IF( NOT DEFINED ${OPTION_NAME} )
		SET( ${OPTION_NAME} ${OPTION_DEFAULT} )
	ENDIF( NOT DEFINED ${OPTION_NAME} )

	SET( ${OPTION_NAME} "${OPTION_DEFAULT}" CACHE ${OPTION_TYPE} "${OPTION_STRING}" )
ENDMACRO()

# Set up library options
# Generic library options
CACHE_VAR_WITH_DEFAULT( ${UPPERCASE_LIBRARY_TARGET_NAME}_BUILD_TYPE "Release" STRING 
	"Choose the type of build, options are: Debug Release RelWithDebInfo MinSizeRel" )
SET( CMAKE_BUILD_TYPE ${${UPPERCASE_LIBRARY_TARGET_NAME}_BUILD_TYPE} CACHE
	INTERNAL "Internalise this variable and manipulate with
	${UPPERCASE_LIBRARY_TARGET_NAME}_BUILD_TYPE" FORCE )
CACHE_VAR_WITH_DEFAULT( ${UPPERCASE_LIBRARY_TARGET_NAME}_INSTALL_PREFIX
	"${CMAKE_INSTALL_PREFIX}" PATH 
	"Install path prefix, prepended onto install directories." )
SET( CMAKE_INSTALL_PREFIX ${${UPPERCASE_LIBRARY_TARGET_NAME}_INSTALL_PREFIX}
	CACHE INTERNAL "Internalise this variable and manipulate with
	${UPPERCASE_LIBRARY_TARGET_NAME}_INSTALL_PREFIX" FORCE )

# Specific library options
OPTION_WITH_DEFAULT( ${UPPERCASE_LIBRARY_TARGET_NAME}_BUILD_EXAMPLE "Build example${CMAKE_EXECUTABLE_SUFFIX} application" ${BUILD_TESTS} )
OPTION_WITH_DEFAULT( ${UPPERCASE_LIBRARY_TARGET_NAME}_BUILD_MINIGZIP "Build minigzip${CMAKE_EXECUTABLE_SUFFIX} application" ${BUILD_TESTS} )

IF( MSVC )
	ADD_DEFINITIONS( -D_CRT_SECURE_NO_WARNINGS )
ENDIF()
IF( BUILD_SHARED_LIBS )
    SET( LIBRARY_INSTALL_TYPE LIBRARY )
	IF( WIN32 )
		SET( LIBRARY_INSTALL_TYPE RUNTIME )
	ENDIF()
ELSE()
	SET( LIBRARY_INSTALL_TYPE ARCHIVE )
ENDIF()

IF( NOT ${UPPERCASE_LIBRARY_TARGET_NAME}_BUILD_EXAMPLE )
	SET( EXCLUDE_EXAMPLE EXCLUDE_FROM_ALL )
ENDIF()
IF(NOT ${UPPERCASE_LIBRARY_TARGET_NAME}_BUILD_MINIGZIP )
	SET( EXCLUDE_MINIGZIP EXCLUDE_FROM_ALL )
ENDIF()

SET( ${UPPERCASE_LIBRARY_TARGET_NAME}_SRCS
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
SET( ${UPPERCASE_LIBRARY_TARGET_NAME}_HDRS
	crc32.h
	deflate.h
	inffast.h
	inffixed.h
	inflate.h
	inftrees.h
	trees.h
	zutil.h )
SET( ${UPPERCASE_LIBRARY_TARGET_NAME}_PUBLIC_HDRS
	zconf.h
	zlib.h )
SET( EXAMPLE_SRCS
	example.c )
SET( MINIGZIP_SRCS
	minigzip.c )

SET( CMAKE_DEBUG_POSTFIX "d" )
SET( CMAKE_MFC_FLAG 0 )

IF( WIN32 AND NOT ${UPPERCASE_LIBRARY_TARGET_NAME}_BUILD_STATIC_LIB )
	SET( ${UPPERCASE_LIBRARY_TARGET_NAME}_WIN32_XTRAS 
		win32/zlib.def
		win32/zlib1.rc )
ENDIF()

IF(${CMAKE_SYSTEM_NAME} STREQUAL "Linux" AND ${CMAKE_SYSTEM_PROCESSOR} STREQUAL "x86_64" )
	SET_SOURCE_FILES_PROPERTIES(${UPPERCASE_LIBRARY_TARGET_NAME}_SRCS
		PROPERTIES COMPILE_FLAGS "-fPIC")
ENDIF()


ADD_LIBRARY( ${LIBRARY_TARGET_NAME} 
	${${UPPERCASE_LIBRARY_TARGET_NAME}_SRCS}
	${${UPPERCASE_LIBRARY_TARGET_NAME}_HDRS}
	${${UPPERCASE_LIBRARY_TARGET_NAME}_PUBLIC_HDRS}
	${${UPPERCASE_LIBRARY_TARGET_NAME}_WIN32_XTRAS} )

if (BUILD_TESTS)
    ADD_EXECUTABLE( example ${EXCLUDE_EXAMPLE} ${EXAMPLE_SRCS} ${${UPPERCASE_LIBRARY_TARGET_NAME}_PUBLIC_HDRS} )
    ADD_EXECUTABLE( minigzip ${EXCLUDE_MINIGZIP} ${MINIGZIP_SRCS} ${${UPPERCASE_LIBRARY_TARGET_NAME}_PUBLIC_HDRS} )
    
    TARGET_LINK_LIBRARIES( example ${LIBRARY_TARGET_NAME} )
    TARGET_LINK_LIBRARIES( minigzip ${LIBRARY_TARGET_NAME} )
endif()

# Install targets
IF( WIN32 AND NOT ${UPPERCASE_LIBRARY_TARGET_NAME}_BUILD_STATIC_LIB )
	SET_TARGET_PROPERTIES(${LIBRARY_TARGET_NAME} PROPERTIES IMPORT_SUFFIX _dll.lib)
	INSTALL( TARGETS ${LIBRARY_TARGET_NAME} ARCHIVE DESTINATION lib )
ENDIF()

INSTALL(TARGETS ${LIBRARY_TARGET_NAME}
    EXPORT zlib-config ${LIBRARY_INSTALL_TYPE}
	DESTINATION lib
	INCLUDES DESTINATION include)
INSTALL(FILES ${${UPPERCASE_LIBRARY_TARGET_NAME}_PUBLIC_HDRS}
	DESTINATION include )

install(EXPORT zlib-config DESTINATION lib/cmake)
include(CMakePackageConfigHelpers)
WRITE_BASIC_PACKAGE_VERSION_FILE(${CMAKE_CURRENT_BINARY_DIR}/zlib-config-version.cmake COMPATIBILITY AnyNewerVersion)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/zlib-config-version.cmake DESTINATION lib/cmake)
