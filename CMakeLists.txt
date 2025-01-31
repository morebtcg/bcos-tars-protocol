#------------------------------------------------------------------------------
# Top-level CMake file for bcos-tars-protocol
# ------------------------------------------------------------------------------
# Copyright (C) 2021 bcos-tars-protocol
# SPDX-License-Identifier: Apache-2.0
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#------------------------------------------------------------------------------

cmake_minimum_required(VERSION 3.10)
set(CMAKE_OSX_DEPLOYMENT_TARGET "11.3" CACHE STRING "Minimum OS X deployment version")

if (NOT DEFINED URL_BASE)
    set(URL_BASE "github.com")
    # set(URL_BASE "github.com.cnpmjs.org")
endif()

include(FetchContent)
# fetch bcos-cmake-scripts
set(BCOS_CMAKE_SCRIPTS_DIR ${CMAKE_CURRENT_BINARY_DIR}/bcos-cmake-scripts)
FetchContent_Declare(bcos-cmake-scripts
    GIT_REPOSITORY https://${URL_BASE}/FISCO-BCOS/bcos-cmake-scripts.git
    GIT_TAG 9b2ac5b979cbc97f640b2285feb2c74a18cfb935
    SOURCE_DIR ${BCOS_CMAKE_SCRIPTS_DIR}
)
FetchContent_MakeAvailable(bcos-cmake-scripts)
set(BCOS_TARS_PROTOCOL_CMAKE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/cmake" CACHE PATH "The cmake path for the library")
list(APPEND CMAKE_MODULE_PATH ${BCOS_CMAKE_SCRIPTS_DIR})
list(APPEND CMAKE_MODULE_PATH ${BCOS_TARS_PROTOCOL_CMAKE_DIR})
#init hunter
include(HunterGate)
HunterGate(
    URL "https://${URL_BASE}/FISCO-BCOS/hunter/archive/203039fdd0826330facd9c3841082dbf35d0c808.tar.gz"
    SHA1 "26ca1840eb26da410913469f2f278569c8d383ae"
    FILEPATH "${BCOS_TARS_PROTOCOL_CMAKE_DIR}/config.cmake"
)
project(bcos-tars-protocol VERSION "3.0.0")
set(VERSION_SUFFIX "-rc1")

# basic settings
include(Options)
configure_project()
include(CompilerSettings)

# copy .clang-format into the project path
include(CopyClangFormat)

include_directories(${CMAKE_INSTALL_INCLUDEDIR})
include_directories(${CMAKE_CURRENT_SOURCE_DIR})
include_directories(${CMAKE_CURRENT_BINARY_DIR})

# install dependencies
include(InstallBcosFrameworkDependencies)
hunter_add_package(tarscpp)
find_package(tarscpp CONFIG REQUIRED)

# for tars generator
set(TARS_HEADER_DIR ${CMAKE_CURRENT_BINARY_DIR}/bcos-tars-protocol/tars)

set(BCOS_TARS_PROTOCOL_TARGET "protocol-tars")
add_subdirectory(bcos-tars-protocol)

if (TESTS)
    # fetch bcos-test
    enable_testing()
    set(CTEST_OUTPUT_ON_FAILURE TRUE)
    add_subdirectory(test)
endif()


include(InstallConfig)
install(
    TARGETS ${BCOS_TARS_PROTOCOL_TARGET}
    EXPORT "${TARGETS_EXPORT_NAME}"
    LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}"
    ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}"
    RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}"
    INCLUDES DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}"
)
# install the include files
include(InstallInclude)

# for doxygen
include(BuildDocs)
buildDoc(bcos-tars-protocol-doc)

# for code coverage
if (COVERAGE)
    include(Coverage)
    config_coverage("tars-protocol-cov" "'/usr*' '${CMAKE_CURRENT_SOURCE_DIR}/bcos-cmake-scripts*' '${CMAKE_CURRENT_SOURCE_DIR}/test/bcos-test*'")
endif ()
