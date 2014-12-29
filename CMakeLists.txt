cmake_minimum_required(VERSION 2.8.7 FATAL_ERROR)

#-----------------------------------------------------------------------
# CPM configuration
#-----------------------------------------------------------------------
set(CPM_MODULE_NAME "es_acorn")
set(CPM_LIB_TARGET_NAME ${CPM_MODULE_NAME})

if ((DEFINED CPM_DIR) AND (DEFINED CPM_UNIQUE_ID) AND (DEFINED CPM_TARGET_NAME))
  set(CPM_LIB_TARGET_NAME ${CPM_TARGET_NAME})
  set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CPM_DIR})
  include(CPM)
else()
  set (CPM_DIR "${CMAKE_CURRENT_BINARY_DIR}/cpm-packages" CACHE TYPE STRING)
  find_package(Git)
  if(NOT GIT_FOUND)
    message(FATAL_ERROR "CPM requires Git.")
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
endif()

#-----------------------------------------------------------------------
# CPM Modules
#-----------------------------------------------------------------------

# ++ MODULE: Entity System
CPM_AddModule("es"
  GIT_REPOSITORY "https://github.com/CIBC-Internal/cpm-entity-system"
  GIT_TAG "origin/SCIRun"
  USE_EXISTING_VER TRUE
  EXPORT_MODULE TRUE)

# ++ MODULE: ES Cereal
CPM_AddModule("es_cereal"
  GIT_REPOSITORY "https://github.com/CIBC-Internal/cpm-es-cereal"
  GIT_TAG "origin/SCIRun"
  EXPORT_MODULE TRUE)

# ++ MODULE: ES Systems
CPM_AddModule("es_systems"
  GIT_REPOSITORY "https://github.com/CIBC-Internal/cpm-es-systems"
  GIT_TAG "origin/SCIRun"
  USE_EXISTING_VER TRUE
  EXPORT_MODULE TRUE)

# I don't want more than one version of this module running around.
CPM_ForceOnlyOneModuleVersion()

# This call will ensure all include directories and definitions are present
# in the target. These correspond to the modules that we added above.
CPM_InitModule(${CPM_MODULE_NAME})

#-----------------------------------------------------------------------
# Source
#-----------------------------------------------------------------------

# Globbing has some downsides, but the advantages outweigh the
# disadvantages.
file (GLOB Sources
  "es-acorn/*.cpp"
  "es-acorn/*.hpp"
  "es-acorn/comp/*.cpp"
  "es-acorn/comp/*.hpp"
  "es-acorn/systems/*.cpp"
  "es-acorn/systems/*.hpp"
  )

#-----------------------------------------------------------------------
# Library setup
#-----------------------------------------------------------------------

# Build the library.
add_library(${CPM_LIB_TARGET_NAME} ${Sources} ${SystemSpecificSources})
if (NOT EMSCRIPTEN AND CPM_LIBRARIES)
  target_link_libraries(${CPM_LIB_TARGET_NAME} ${CPM_LIBRARIES})
endif()
