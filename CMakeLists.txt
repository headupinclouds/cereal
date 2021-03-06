cmake_minimum_required (VERSION 3.0)
project (cereal VERSION 1.1.2)

option(SKIP_PORTABILITY_TEST "Skip portability tests" OFF)

set(CMAKE_CXX_FLAGS "-std=c++11 -Wall -Werror -g -Wextra -Wshadow -pedantic ${CMAKE_CXX_FLAGS}")

include_directories(./include)

find_package(Boost COMPONENTS serialization unit_test_framework)

if(Boost_FOUND)
  include_directories(${Boost_INCLUDE_DIRS})
  enable_testing()
  add_subdirectory(unittests)
endif(Boost_FOUND)

add_subdirectory(sandbox)

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
