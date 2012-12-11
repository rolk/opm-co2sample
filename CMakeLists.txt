# -*- mode: cmake; tab-width: 2; indent-tabs-mode: t; truncate-lines: t; compile-command: "cmake -Wdev" -*-
# vim: set filetype=cmake autoindent tabstop=2 shiftwidth=2 noexpandtab softtabstop=2 nowrap
cmake_minimum_required (VERSION 2.8)
project (opm-co2sample)
enable_language (CXX)

# build debug by default
if (NOT CMAKE_CONFIGURATION_TYPES AND NOT CMAKE_BUILD_TYPE)
	set (CMAKE_BUILD_TYPE "Debug")
endif (NOT CMAKE_CONFIGURATION_TYPES AND NOT CMAKE_BUILD_TYPE)
message (STATUS "Build type: ${CMAKE_BUILD_TYPE}")

# extensions to the build system
list (APPEND CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake/Modules")

# always build with debug info
include (UseDebugSymbols)
include (UseWarnings)

# use optimizations if we are doing a release build
include (UseOptimization)

# use tricks to build faster
include (UseFastBuilds)

# if Findopm-core.cmake is not found in CMAKE_MODULE_PATH, then a cached
# variable called opm-core_DIR will be created to search for the path of
# the file opm-core-config.cmake which should be located in the project
if (NOT DEFINED opm-core_DIR)
	set (opm-core_DIR
		"${PROJECT_BINARY_DIR}/../opm-core" CACHE LOCATION
		"Build tree of opm-core"
		)
endif (NOT DEFINED opm-core_DIR)
include (OpmFind)
find_and_append_package (opm-core REQUIRED)
include_directories (${opm-co2sample_INCLUDE_DIRS})
link_directories (${opm-co2sample_LIBRARY_DIRS})
add_definitions (${opm-co2sample_DEFINITIONS})

# configuration defines necessary for the header files
include (ConfigVars)
configure_vars (
	FILE  CXX  "${PROJECT_BINARY_DIR}/config.h"
	WRITE ${opm-co2sample_CONFIG_VARS}
	)

# create executables in this directory
set (CMAKE_RUNTIME_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}/bin")

# locate source files that makes out program(s)
file (GLOB_RECURSE co2_3d_SOURCES "3d/*.cpp")

# setup compilation of this particular executable
add_executable (co2_3d ${co2_3d_SOURCES})
target_link_libraries (co2_3d ${opm-co2sample_LIBRARIES})

# queue this executable to be stripped
strip_debug_symbols (RUNTIME co2_3d)

# look for the vertical equilibrium library
if (NOT DEFINED opm-verteq_DIR)
	set (opm-verteq_DIR
		"${PROJECT_BINARY_DIR}/../opm-verteq" CACHE LOCATION
		"Build tree of opm-verteq"
		)
endif (NOT DEFINED opm-verteq_DIR)
find_and_append_package (opm-verteq REQUIRED)
include_directories (${opm-co2sample_INCLUDE_DIRS})
link_directories (${opm-co2sample_LIBRARY_DIRS})
add_definitions (${opm-co2sample_DEFINITIONS})

# locate source files that makes out program(s)
file (GLOB_RECURSE co2_2d_SOURCES "2d/*.cpp")
add_executable (co2_2d ${co2_2d_SOURCES})
target_link_libraries (co2_2d ${opm-co2sample_LIBRARIES})

# queue this executable to be stripped
strip_debug_symbols (RUNTIME co2_2d)

# example input file (realize directory names)
configure_file (
  ${PROJECT_SOURCE_DIR}/data/tube.param.in
  ${PROJECT_BINARY_DIR}/bin/tube.param
  )
