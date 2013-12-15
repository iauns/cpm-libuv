# We require 2.8.9 due to bugs in CMake's external projects in 2.8.7.
cmake_minimum_required(VERSION 2.8.9 FATAL_ERROR)

#-----------------------------------------------------------------------
# CPM configuration
#-----------------------------------------------------------------------
set(CPM_MODULE_NAME libuv)
set(CPM_LIB_TARGET_NAME ${CPM_MODULE_NAME})

if ((DEFINED CPM_DIR) AND (DEFINED CPM_UNIQUE_ID) AND (DEFINED CPM_TARGET_NAME))
  set(CPM_LIB_TARGET_NAME ${CPM_TARGET_NAME})
  set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CPM_DIR})
  include(CPM)
else()
  set(CPM_DIR "${CMAKE_CURRENT_BINARY_DIR}/cpm-packages" CACHE TYPE STRING)
  find_package(Git)
  if(NOT GIT_FOUND)
    message(FATAL_ERROR "CPM requires Git.")
  endif()
  if (NOT EXISTS ${CPM_DIR}/CPM.cmake)
    execute_process(
      COMMAND "${GIT_EXECUTABLE}" clone https://github.com/iauns/cpm ${CPM_DIR}
      RESULT_VARIABLE error_code
      OUTPUT_VARIABLE head_sha)
    if(error_code)
      message(FATAL_ERROR "CPM failed to get the hash for HEAD")
    endif()
  endif()
  include(${CPM_DIR}/CPM.cmake)
endif()

# All externals *must* define this.
CPM_ForceOnlyOneModuleVersion()

CPM_InitModule(${CPM_MODULE_NAME})

#------------------------------------------------------------------------------
# External project
#------------------------------------------------------------------------------

if (UNIX)
  include(ExternalProject)
  set(EP_TARGET_NAME LIBUV_LIB_EP)
  ExternalProject_Add(${EP_TARGET_NAME}
    GIT_REPOSITORY "https://github.com/joyent/libuv.git"
    GIT_TAG "v0.11.16"
    CONFIGURE_COMMAND sh <SOURCE_DIR>/autogen.sh && ./configure --prefix=${CMAKE_CURRENT_SOURCE_DIR}
    PATCH_COMMAND ""
    INSTALL_DIR ""
    BUILD_COMMAND "make"
    BUILD_IN_SOURCE 1
    INSTALL_COMMAND "make" "install"
    )

  set(EP_LIBRARY ${CPM_LIB_TARGET_NAME})
  add_library(${EP_LIBRARY} SHARED IMPORTED GLOBAL)
  add_dependencies(${EP_LIBRARY} ${EP_TARGET_NAME})

  # Grab the source directory, that is where all of the library files
  # will reside. We will staically link against them.
  ExternalProject_Get_Property(${EP_TARGET_NAME} SOURCE_DIR)

  # Generate a library path and set the imported location for mongoc.
  set_target_properties(${EP_LIBRARY}
    PROPERTIES
    IMPORTED_LOCATION ${CMAKE_CURRENT_SOURCE_DIR}/lib/${CMAKE_STATIC_LIBRARY_PREFIX}uv${CMAKE_STATIC_LIBRARY_SUFFIX}
    )

  # Grab the source directory and set appropriate include dir.
  ExternalProject_Get_Property(${EP_TARGET_NAME} SOURCE_DIR)
  CPM_ExportAdditionalIncludeDir("${SOURCE_DIR}/include")
else()
  if (WIN32)
    # TODO: Ensure python is present on the system and use gyp to build.
    message(FATAL_ERROR "Windows is supported by libuv, but it is not implemented in CPM's libuv external yet.")
  else()
    message(FATAL_ERROR "CPM libuv: Unrecognized platform.")
  endif()
endif()

