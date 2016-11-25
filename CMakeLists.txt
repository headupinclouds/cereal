cmake_minimum_required (VERSION 3.0) # hunter minimum
project (cereal VERSION 1.2.1)

option(SKIP_PORTABILITY_TEST "Skip portability tests" OFF)
if(NOT CMAKE_VERSION VERSION_LESS 3.0) # installing cereal requires INTERFACE lib
    option(JUST_INSTALL_CEREAL "Don't do anything besides installing the library" OFF)
endif()

if(NOT CMAKE_VERSION VERSION_LESS 3.0)
    add_library(cereal INTERFACE)
    target_include_directories(cereal INTERFACE
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>
        )

      ### Install ###
      # Note: I'm opting for boilerplate hunter install and package config to make
      # sure hunter compatibility is preserved.  This looks very close to the
      # upstream CMAke.
      
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
        DIRECTORY "include/cereal"
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
endif()

if(JUST_INSTALL_CEREAL)
    return()
  endif()

option(THREAD_SAFE "Use mutexes to ensure thread safety" OFF)
if(THREAD_SAFE)
    target_compile_definitions(cereal PUBLIC CEREAL_THREAD_SAFE=1)
    set(CEREAL_THREAD_LIBS "pthread")
else()
    set(CEREAL_THREAD_LIBS "")
endif()

if(NOT MSVC)
    set(CMAKE_CXX_FLAGS "-Wall -g -Wextra -Wshadow -pedantic ${CMAKE_CXX_FLAGS}")
    option(WITH_WERROR "Compile with '-Werror' C++ compiler flag" ON)
    if(WITH_WERROR)
        set(CMAKE_CXX_FLAGS "-Werror ${CMAKE_CXX_FLAGS}")
    endif(WITH_WERROR)
    if(CMAKE_VERSION VERSION_LESS 3.1)
        set(CMAKE_CXX_FLAGS "-std=c++11 ${CMAKE_CXX_FLAGS}")
    else()
        if(NOT DEFINED CMAKE_CXX_STANDARD OR CMAKE_CXX_STANDARD STREQUAL "98")
            set(CMAKE_CXX_STANDARD 11)
        endif()
        set(CMAKE_CXX_STANDARD_REQUIRED ON)
    endif()
endif()

include_directories(./include)

find_package(Boost COMPONENTS serialization unit_test_framework)

if(Boost_FOUND)
  include_directories(SYSTEM ${Boost_INCLUDE_DIRS})
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
