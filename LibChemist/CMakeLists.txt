cmake_minimum_required(VERSION 3.2)
project(${CODE_NAME}-Core CXX)
include(${${CODE_NAME}_ROOT}/cmake/cmake_macros.cmake)
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_EXTENSIONS False)

# CMake doesn't support Intel CXX standard until cmake 3.6
if("${CMAKE_CXX_COMPILER_ID}" MATCHES "Intel")
  if("${CMAKE_VERSION}" VERSION_LESS "3.6")
      add_compile_options(-std=c++${CMAKE_CXX_STANDARD})
  endif()
endif()
set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -Wall")
my_add_subdirectory(lut)
include_directories(${${CODE_NAME}_ROOT})

add_library(${CODE_NAME} ${lut_SRC}
                         Atom.cpp
                         BasisSet.cpp
                         BasisSetParser.cpp
                         BasisShell.cpp
                         SetOfAtoms.cpp
                         SetOfAtomsParser.cpp
                         ShellTypes.cpp
)
install(TARGETS ${CODE_NAME}
        LIBRARY DESTINATION lib/
        ARCHIVE DESTINATION lib/
        RUNTIME DESTINATION lib/)

# Install all headers
install(DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}"
        DESTINATION include
        FILES_MATCHING
        REGEX ".*\.h(pp)?$")

# Write the configure file so other packages can find us
configure_file("${${CODE_NAME}_ROOT}/cmake/${CODE_NAME}Config.cmake.in"
               ${CODE_NAME}Config.cmake @ONLY)

# Install said file
install(FILES ${CMAKE_BINARY_DIR}/${CODE_NAME}Config.cmake
        DESTINATION share/cmake/${CODE_NAME})
