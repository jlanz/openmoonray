# Copyright 2023 DreamWorks Animation LLC
# SPDX-License-Identifier: Apache-2.0

cmake_minimum_required (VERSION 3.23.1)

project(openmoonray)

list(APPEND CMAKE_MESSAGE_CONTEXT ${PROJECT_NAME})
if(NOT DEFINED CMAKE_INSTALL_PREFIX OR ${CMAKE_INSTALL_PREFIX} STREQUAL "/usr/local")
    set(CMAKE_INSTALL_PREFIX ${CMAKE_SOURCE_DIR}/release)
endif()
list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake_modules/cmake)

# Make sure Cuda's bin directory is in ${PATH} so that check_language can find it.
set(ENV{PATH} "${CUDAToolkit_ROOT}/bin:$ENV{PATH}")

include(CheckLanguage)
check_language(CUDA)
if(CMAKE_CUDA_COMPILER)
    enable_language(CUDA)
else()
    message(STATUS "No CUDA support")
endif()

include(CTest)
include(GNUInstallDirs)

option(BUILD_QT_APPS             "Whether or not to build apps that require Qt" YES)
option(MOONRAY_USE_CUDA          "Whether to use CUDA (and Optix) - Enables XPU mode and Optix denoising" YES)

# if OPT_LEVEL is set use its value to override the CMAKE_BUILD_TYPE because the
# rez cmake plugin does not have an easy way to set the build_target.
set(opt_level $ENV{OPT_LEVEL})
if(opt_level STREQUAL opt-debug)
    set(CMAKE_BUILD_TYPE RelWithDebInfo)
elseif(opt_level STREQUAL debug)
    set(CMAKE_BUILD_TYPE Debug)
elseif(opt_level STREQUAL opt)
    set(CMAKE_BUILD_TYPE Release)
endif()

# default to Release if no build type specified
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release)
endif()

add_subdirectory(arras)
add_subdirectory(moonray)
add_subdirectory(scripts)

if(GLD STREQUAL "$ENV{STUDIO}")
    install(FILES
        ${CMAKE_CURRENT_SOURCE_DIR}/SDKScript
        DESTINATION
        .)
endif()
