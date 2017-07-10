cmake_minimum_required(VERSION 3.1)
set(CODE_NAME LibChemist)
project(${CODE_NAME}-superbuild CXX)
include(ExternalProject)
list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)
include(cmake/cmake_macros.cmake)

#Debug or Release build?
option_w_default(CMAKE_BUILD_TYPE "Release")
set(STAGE_DIR ${CMAKE_BINARY_DIR}/stage)

ExternalProject_Add(${CODE_NAME}
    SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/${CODE_NAME}
    CMAKE_ARGS -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
               -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
               -DCMAKE_CXX_COMPILER=${CMAKE_CXX_COMPILER}
               -D${CODE_NAME}_ROOT=${CMAKE_CURRENT_SOURCE_DIR}
               -DCODE_NAME=${CODE_NAME}
    BUILD_ALWAYS 1
    INSTALL_COMMAND ${CMAKE_MAKE_PROGRAM} install DESTDIR=${STAGE_DIR}
    CMAKE_CACHE_ARGS -DCMAKE_PREFIX_PATH:LIST=${CMAKE_PREFIX_PATH}
                     -DCMAKE_INSTALL_RPATH:LIST=${CMAKE_INSTALL_RPATH}
                     -DCMAKE_CXX_FLAGS:STRING=${CMAKE_CXX_FLAGS}
                     -DCMAKE_MODULE_PATH:LIST=${CMAKE_MODULE_PATH}
)

###############################
# Tests
###############################

set(${CODE_NAME}_PREFIX ${STAGE_DIR}${CMAKE_INSTALL_PREFIX})
list(APPEND CMAKE_PREFIX_PATH ${${CODE_NAME}_PREFIX})
ExternalProject_Add(${CODE_NAME}_test
    SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/${CODE_NAME}-test
    CMAKE_ARGS -DCMAKE_INSTALL_PREFIX=${CMAKE_BINARY_DIR}/test_stage
               -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
               -DCMAKE_CXX_COMPILER=${CMAKE_CXX_COMPILER}
               -DSTAGE_DIR=${STAGE_DIR}
               -DCODE_NAME=${CODE_NAME}
               -D${CODE_NAME}_PREFIX=${${CODE_NAME}_PREFIX}
    BUILD_ALWAYS 1
    CMAKE_CACHE_ARGS -DCMAKE_PREFIX_PATH:LIST=${CMAKE_PREFIX_PATH}
                     -DCMAKE_INSTALL_RPATH:LIST=${CMAKE_INSTALL_RPATH}
                     -DCMAKE_CXX_FLAGS:STRING=${CMAKE_CXX_FLAGS}
                     -DCMAKE_MODULE_PATH:LIST=${CMAKE_MODULE_PATH}
)
add_dependencies(${CODE_NAME}_test ${CODE_NAME})


# This file will allow us to run ctest in the top-level build dir
# Basically it just defers to the actual top-level CTestTestfile.cmake in the
# build directory for this project
file(WRITE ${CMAKE_BINARY_DIR}/CTestTestfile.cmake "subdirs(test_stage)")

# Install the staging directory
install(DIRECTORY ${CMAKE_BINARY_DIR}/stage/${CMAKE_INSTALL_PREFIX}/
        DESTINATION ${CMAKE_INSTALL_PREFIX} USE_SOURCE_PERMISSIONS)

#Install our header files
install(DIRECTORY "cmake"
        DESTINATION share/cmake
        FILES_MATCHING REGEX ".*\.cmake")

#####################
# uninstall target
#####################
configure_file(
"${CMAKE_CURRENT_SOURCE_DIR}/cmake/cmake_uninstall.cmake.in"
"${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
IMMEDIATE @ONLY)

add_custom_target(uninstall
  COMMAND ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake)
