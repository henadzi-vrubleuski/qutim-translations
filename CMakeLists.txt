cmake_minimum_required (VERSION 2.6 FATAL_ERROR)

project(translations)

if(${CMAKE_CURRENT_SOURCE_DIR} STREQUAL ${CMAKE_SOURCE_DIR} )
        set(QUTIM_PATH "" CACHE FILEPATH "Path to qutim sources")
        set(QUTIM_BINARY_DIR "" CACHE FILEPATH "Path to qutim build directory")
        set(QUTIM_LIBRARY QUTIM_LIBRARY-NOTFOUND CACHE FILEPATH "Path to qutim library")
endif ( ${CMAKE_CURRENT_SOURCE_DIR} STREQUAL ${CMAKE_SOURCE_DIR} )
if( QUTIM_BINARY_DIR )
        set( QUTIM_INCLUDE_DIR ${QUTIM_BINARY_DIR}/libqutim/include )
        set( QUTIM_LIBRARY_DIR ${QUTIM_BINARY_DIR}/libqutim/ )
endif( QUTIM_BINARY_DIR )

LIST(APPEND CMAKE_MODULE_PATH "cmake")
LIST(APPEND CMAKE_MODULE_PATH "${QUTIM_PATH}/cmake")

include(MacroEnsureVersion)
include(QutIMMacros)
find_package(QutIM REQUIRED)

find_package(Qt4 REQUIRED)

set(LANG_DIR "${CMAKE_CURRENT_BINARY_DIR}/languages")
file(MAKE_DIRECTORY ${LANG_DIR} )
add_custom_target( translations ALL echo DEPENDS ${TRANSLATIONS_BINARY} )

set( basePath "${CMAKE_CURRENT_SOURCE_DIR}/modules" )
file(GLOB MODULES RELATIVE "${basePath}" "${basePath}/*/*.po" )
foreach( moduleFile ${MODULES} )
	string( REGEX REPLACE "/.*" "" module ${moduleFile} )
	string( REGEX REPLACE ".*/(.*)\\.po" "\\1" language ${moduleFile} )
	message( "${moduleFile} ${module} ${language}" )
	file(MAKE_DIRECTORY "${LANG_DIR}/${language}")
	add_custom_command(
		TARGET    translations
		DEPENDS   ${poFile}
		COMMAND   ${QT_LRELEASE_EXECUTABLE}
		ARGS      -silent "${basePath}/${moduleFile}" -qm "${LANG_DIR}/${language}/${module}.qm"
	)
endforeach()

QUTIM_ADD_ARTWORK(${LANG_DIR})
