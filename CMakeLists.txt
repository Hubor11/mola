# ------------------------------------------------------------------------------
#        A Modular Optimization framework for Localization and mApping
#                               (MOLA)
#
# Copyright (C) 2018-2021, Jose Luis Blanco-Claraco, contributors (AUTHORS.md)
# All rights reserved.
# Released under GNU GPL v3. See LICENSE file
# ------------------------------------------------------------------------------

# Minimum CMake version, limited by:
#   cxx_std_17 feature (>=3.9)
cmake_minimum_required(VERSION 3.9)

# Tell CMake we'll use C++ for use in its tests/flags
project(mola LANGUAGES CXX)

option(BUILD_SHARED_LIBS "Build shared libraries" ON)

# Enforce selecting one of a set of valid cmake build types:
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
	set(default_build_type "Release")
	message(STATUS "Setting build type to '${default_build_type}' as none was specified.")
	set(CMAKE_BUILD_TYPE "${default_build_type}" CACHE
		STRING "Choose the type of build." FORCE)
endif()
# Set the possible values of build type for cmake-gui
if (DEFINED CACHE{CMAKE_BUILD_TYPE})
	set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS
		"Debug" "Release" "MinSizeRel" "RelWithDebInfo" "Coverage" "SanitizeAddress" "SanitizeThread")
endif()

# Include std cmake scripts:
include(CheckCXXCompilerFlag) # for check_cxx_compiler_flag()
include(CMakeDetermineCXXCompiler)

# Allow finding cmake packages that we are just configuring now:
list(APPEND CMAKE_MODULE_PATH ${CMAKE_BINARY_DIR})

# Common CMake scripts:
add_subdirectory(modules/mola-common)

# this is to ensure cmake finds mola-common-config.cmake:
set(mola-common_DIR ${CMAKE_BINARY_DIR})
# find here so external projects can use it
find_package(mola-common REQUIRED)

# GTSAM:
find_package(GTSAM)
find_package(GTSAM_UNSTABLE)
if (GTSAM_FOUND AND GTSAM_UNSTABLE_FOUND)
	message(STATUS "GTSAM found version: ${GTSAM_VERSION}")
else()
  message(FATAL_ERROR
    " *** GTSAM & GTSAM_UNSTABLE>=4.0.2 was not found ***\n"
    "Please, install GTSAM or build from sources and set the cmake variable GTSAM_DIR to the build directory.\n"
    "In Ubuntu, the easiest solution is to install gtsam from: https://launchpad.net/~joseluisblancoc/+archive/ubuntu/gtsam-develop"
  )
endif()

# Process subdirectories:
option(MOLA_ENABLE_UNIT_TESTS "Build unit tests" ON)
if(MOLA_ENABLE_UNIT_TESTS)
    include(GoogleTest)
    enable_testing()
endif()

# External projects:
add_subdirectory(externals)

# MOLA modules:
add_subdirectory(modules)

# Docs: Don't build using CMake, but instead by means of a custom docs/Makefile script

message(STATUS "========================= MOLA configuration ======================")
message(STATUS "C++ Compiler              : ${CMAKE_CXX_COMPILER} Version: ${CMAKE_CXX_COMPILER_VERSION}")
message(STATUS "CMake version             : ${CMAKE_VERSION}")
message(STATUS "CMake generator           : ${CMAKE_GENERATOR}")
message(STATUS "CMake build tool          : ${CMAKE_BUILD_TOOL}")
message(STATUS "Sphinx (for docs)         : ${SPHINX_EXECUTABLE}")
message(STATUS "===================================================================")
