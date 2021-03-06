# Copyright (c) 2013-2017, EPFL/Blue Brain Project
#                          Daniel Nachbaur <daniel.nachbaur@epfl.ch>
#
# This file is part of Brion <https://github.com/BlueBrain/Brion>
#

cmake_minimum_required(VERSION 3.1 FATAL_ERROR)
project(Brion VERSION 3.0.0)
set(Brion_VERSION_ABI 9)

list(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/CMake
  ${CMAKE_SOURCE_DIR}/CMake/common ${PROJECT_SOURCE_DIR}/Pydoxine/CMake)
if(NOT EXISTS ${CMAKE_SOURCE_DIR}/CMake/common/Common.cmake)
  message(FATAL_ERROR "CMake/common missing, run: git submodule update --init")
endif()

set(BRION_DESCRIPTION "The Blue Brain C++ I/O library")
set(BRION_MAINTAINER_NAME "Blue Brain Project")
set(BRION_MAINTAINER_EMAIL "<bbp-open-source@googlegroups.com>")
set(BRION_MAINTAINER "${BRION_MAINTAINER_NAME} ${BRION_MAINTAINER_EMAIL}")
set(BRION_LICENSE LGPL)
set(BRION_DEB_DEPENDS libhdf5-serial-dev libboost-date-time-dev
  libboost-filesystem-dev libboost-iostreams-dev libboost-regex-dev
  libboost-system-dev libboost-test-dev libboost-thread-dev
  python3-sphinx python3-lxml python3-dev python3-numpy)
set(BRION_PORT_DEPENDS sphinx py-lxml)

set(COMMON_PROJECT_DOMAIN ch.epfl.bluebrain)
include(Common)

common_find_package(BBPTestData)
common_find_package(Boost REQUIRED COMPONENTS
  date_time filesystem iostreams program_options regex system unit_test_framework)
common_find_package(Boost COMPONENTS python${USE_BOOST_PYTHON_VERSION})
common_find_package(HighFive REQUIRED)
common_find_package(Keyv REQUIRED)
common_find_package(Lunchbox REQUIRED)
common_find_package(MVDTool)
common_find_package(NumPy 1.7)
common_find_package(OpenMP)
common_find_package(PythonInterp)
common_find_package(PythonLibs)
common_find_package(Servus REQUIRED)
common_find_package(Sphinx 1.3)
common_find_package(vmmlib REQUIRED)
option(BRION_USE_ZEROEQ "Use ZeroEQ for plugin backend and morphologyServer" OFF)
if(BRION_USE_ZEROEQ)
  git_subproject(ZeroEQ https://github.com/HBPVIS/ZeroEQ.git 1e66ee3)
  common_find_package(ZeroEQ REQUIRED)
endif()
common_find_package_post()

list(APPEND BRION_DEPENDENT_LIBRARIES Boost Servus vmmlib)

add_subdirectory(brion)
add_subdirectory(brain)
add_subdirectory(apps)
add_subdirectory(tests)

include(CPackConfig)

set(DOXYGEN_MAINPAGE_MD README.md)
set(DOXYGEN_EXPAND_AS_DEFINED "BRAIN_API BRION_API")
set(DOXYGEN_EXTRA_INPUT "${PROJECT_SOURCE_DIR}/README.md ${CMAKE_INSTALL_PREFIX}/include/brain")
set(DOXYGEN_PREDEFINED_MACROS "BRAIN_API:=")
include(DoxygenRule)

add_subdirectory(doc/python)

# fill out setup.py metadata for wheel distribution
configure_file(packaging/python_wheels/setup.py.in
               ${PROJECT_BINARY_DIR}/packaging/python_wheels/setup.py)
