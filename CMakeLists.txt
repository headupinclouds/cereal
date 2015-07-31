# Copyright (c) 2014, Randolph Voorhies, Shane Grant
# Copyright (C) 2015 Ruslan Baratov
# Copyright (C) 2015 David Hirvonen
# License: Boost Software License
# See LICENSE.txt for the full license.

#
# Provide interface lib for clean package config use
#

cmake_minimum_required (VERSION 2.6.2)
project (cereal VERSION 1.1.2)

add_library(cereal INTERFACE)

### Install ###
set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")

set(config_install_dir "lib/cmake/${PROJECT_NAME}")
set(include_install_dir "include")

set(version_config "${generated_dir}/${PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(targets_export_name "${PROJECT_NAME}Targets")
set(namespace "${PROJECT_NAME}::")

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    "${version_config}" COMPATIBILITY SameMajorVersion
)

# Note: use 'targets_export_name'
configure_file("cmake/Config.cmake.in" "${project_config}" @ONLY)

install(
    TARGETS cereal
    EXPORT "${targets_export_name}"
    INCLUDES DESTINATION "${include_install_dir}"
)

install(
    DIRECTORY "cereal"
    DESTINATION "${include_install_dir}"
)

install(
    FILES "${project_config}" "${version_config}"
    DESTINATION "${config_install_dir}"
)

install(
    EXPORT "${targets_export_name}"
    NAMESPACE "${namespace}"
    DESTINATION "${config_install_dir}"
)

#
# Begining of the standard (non-hunterized) CMakeLists
#

option(SKIP_PORTABILITY_TEST "Skip portability tests" OFF)
option(BUILD_TESTS "Build the unit tests" OFF)
option(BUILD_DOCUMENTATION "Build the documentations" OFF)


if(BUILD_TESTS)
  set(CMAKE_CXX_FLAGS "-std=c++11 -Wall -Werror -g -Wextra -Wshadow -pedantic ${CMAKE_CXX_FLAGS}")

  include_directories(./include)

  find_package(Boost COMPONENTS serialization unit_test_framework)

  if(Boost_FOUND)
    include_directories(${Boost_INCLUDE_DIRS})
    enable_testing()
    add_subdirectory(unittests)
  endif(Boost_FOUND)

  add_subdirectory(sandbox)
endif(BUILD_TESTS)


if(BUILD_DOCUMENTATION)
  find_package(Doxygen)
  if(DOXYGEN_FOUND)

    configure_file("${CMAKE_CURRENT_SOURCE_DIR}/doc/doxygen.in" "${CMAKE_CURRENT_BINARY_DIR}/doxygen.cfg" @ONLY)
    add_custom_target(doc
      COMMAND ${DOXYGEN_EXECUTABLE} "${CMAKE_CURRENT_BINARY_DIR}/doxygen.cfg"
      WORKING_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}"
      COMMENT "Generating API documentation with Doxygen" VERBATIM
      )

    configure_file("${CMAKE_CURRENT_SOURCE_DIR}/scripts/updatedoc.in" "${CMAKE_CURRENT_BINARY_DIR}/updatedoc.sh" @ONLY)
    add_custom_target(update-doc
      COMMAND "${CMAKE_CURRENT_BINARY_DIR}/updatedoc.sh"
      DEPENDS doc
      COMMENT "Copying documentation to gh-pages branch" VERBATIM
      )

  endif(DOXYGEN_FOUND)
endif(BUILD_DOCUMENTATION)
