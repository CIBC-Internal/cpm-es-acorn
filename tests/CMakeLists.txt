if(APPLE)
  cmake_minimum_required(VERSION 2.8.11 FATAL_ERROR)
else()
  cmake_minimum_required(VERSION 2.8.7 FATAL_ERROR)
endif()

project(gcore_tests)

# Turn on verbose output from es-cereal. This just manifests in warnings
# when we don't find a particular name in a serialized structure.
# This will arise when using delta compression schemes that only serialize
# the absolutely necessary data.
add_definitions(-DCPM_ES_CEREAL_VERBOSE_OUTPUT)

#-----------------------------------------------------------------------
# C++11
#-----------------------------------------------------------------------
if (UNIX)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
  if (APPLE)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -stdlib=libc++")
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -stdlib=libc++")
  endif ()
endif ()

#------------------------------------------------------------------------------
# Required CPM Setup - See: http://github.com/CIBC-Internal/cpm
#------------------------------------------------------------------------------
set(CPM_DIR "${CMAKE_CURRENT_BINARY_DIR}/cpm-packages" CACHE TYPE STRING)
find_package(Git)
if(NOT GIT_FOUND)
  message(FATAL_ERROR "CPM requires Git.")
endif()
if ((NOT DEFINED CPM_MODULE_CACHE_DIR) AND (NOT "$ENV{CPM_CACHE_DIR}" STREQUAL ""))
  set(CPM_MODULE_CACHE_DIR "$ENV{CPM_CACHE_DIR}")
endif()
if ((NOT EXISTS ${CPM_DIR}/CPM.cmake) AND (DEFINED CPM_MODULE_CACHE_DIR))
  if (EXISTS "${CPM_MODULE_CACHE_DIR}/github_cibcinternal_cpm")
    message(STATUS "Found cached version of CPM.")
    file(COPY "${CPM_MODULE_CACHE_DIR}/github_cibcinternal_cpm/" DESTINATION ${CPM_DIR})
  endif()
endif()
if (NOT EXISTS ${CPM_DIR}/CPM.cmake)
  message(STATUS "Cloning repo (https://github.com/CIBC-Internal/cpm)")
  execute_process(
    COMMAND "${GIT_EXECUTABLE}" clone https://github.com/CIBC-Internal/cpm ${CPM_DIR}
    RESULT_VARIABLE error_code
    OUTPUT_QUIET ERROR_QUIET)
  if(error_code)
    message(FATAL_ERROR "CPM failed to get the hash for HEAD")
  endif()
endif()
include(${CPM_DIR}/CPM.cmake)

# ++ MODULE: General components and systems.
CPM_AddModule("es_acorn"
  SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/..")

# ++ EXTERNAL-MODULE: Google Test
CPM_AddModule("google_test"
  GIT_REPOSITORY "https://github.com/CIBC-Internal/cpm-google-test"
  GIT_TAG "origin/master"
  USE_EXISTING_VER TRUE)

CPM_Finish()

#-----------------------------------------------------------------------
# Setup strict warnings and werror
#-----------------------------------------------------------------------

if(APPLE)
  set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -Werror -Wshadow")
endif()

#-----------------------------------------------------------------------
# Setup source
#-----------------------------------------------------------------------
# Setup spire test sources. Even though this is a batch renderer, this is
# a good test of concept.

file(GLOB Sources
  "*.cpp"
  "*.hpp"
  )

########################################################################
# Setup executable

# For google test...
if (UNIX)
  if (NOT APPLE)
    set(PTHREADS "-pthread")
  endif()
endif()

add_executable(gcore_tests ${Sources})
target_link_libraries(gcore_tests 
  ${CPM_LIBRARIES}
  ${PTHREADS})
add_dependencies(gcore_tests ${CPM_LIBRARIES})

