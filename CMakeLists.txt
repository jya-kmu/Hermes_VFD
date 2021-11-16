# CMakeLists files in this project can
# refer to the root source directory of the project as ${HERMES_SOURCE_DIR} and
# to the root binary directory of the project as ${HERMES_BINARY_DIR}.
cmake_minimum_required (VERSION 3.10)

if(CMAKE_SIZEOF_VOID_P LESS 8)
  message(FATAL_ERROR "Hermes currently requires a 64-bit processor")
endif()

project(HERMES_VFD)

include_directories(${PROJECT_BINARY_DIR})

if(NOT CMAKE_INSTALL_RPATH_USE_LINK_PATH)
  set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)
endif()

#------------------------------------------------------------------------------
# Set module path
#------------------------------------------------------------------------------
set(HERMES_VFD_CMAKE_MODULE_PATH "${HERMES_VFD_SOURCE_DIR}/CMake")
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${HERMES_VFD_CMAKE_MODULE_PATH})

#------------------------------------------------------------------------------
# Version information
#------------------------------------------------------------------------------
set(HERMES_VFD_VERSION_MAJOR "0")
set(HERMES_VFD_VERSION_MINOR "1")
set(HERMES_VFD_VERSION_PATCH "0")
set(HERMES_VFD_PACKAGE "hermes_vfd")
set(HERMES_VFD_PACKAGE_NAME "HERMES_VFD")
set(HERMES_VFD_PACKAGE_VERSION "${HERMES_VFD_VERSION_MAJOR}.${HERMES_VFD_VERSION_MINOR}.${HERMES_VFD_VERSION_PATCH}")
set(HERMES_VFD_PACKAGE_VERSION_MAJOR "${HERMES_VFD_VERSION_MAJOR}.${HERMES_VFD_VERSION_MINOR}")
set(HERMES_VFD_PACKAGE_VERSION_MINOR "${HERMES_VFD_VERSION_PATCH}")
set(HERMES_VFD_PACKAGE_STRING "${HERMES_VFD_PACKAGE_NAME} ${HERMES_VFD_PACKAGE_VERSION}")
set(HERMES_VFD_PACKAGE_TARNAME "${HERMES_VFD_PACKAGE}")

#------------------------------------------------------------------------------
# Setup install and output Directories
#------------------------------------------------------------------------------
if(NOT HERMES_VFD_INSTALL_BIN_DIR)
  set(HERMES_VFD_INSTALL_BIN_DIR ${CMAKE_INSTALL_PREFIX}/bin)
endif()
if(NOT HERMES_VFD_INSTALL_LIB_DIR)
  set(HERMES_VFD_INSTALL_LIB_DIR ${CMAKE_INSTALL_PREFIX}/lib)
endif()
if(NOT HERMES_VFD_INSTALL_INCLUDE_DIR)
  set(HERMES_VFD_INSTALL_INCLUDE_DIR ${CMAKE_INSTALL_PREFIX}/include)
endif()
if(NOT HERMES_VFD_INSTALL_DATA_DIR)
  set(HERMES_VFD_INSTALL_DATA_DIR ${CMAKE_INSTALL_PREFIX}/share)
endif()

#------------------------------------------------------------------------------
if(NOT HERMES_VFD_EXTERNALLY_CONFIGURED)
  set(CMAKE_RUNTIME_OUTPUT_DIRECTORY
      ${PROJECT_BINARY_DIR}/bin CACHE PATH "Single Directory for all Executables."
  )
  set(EXECUTABLE_OUTPUT_PATH ${CMAKE_RUNTIME_OUTPUT_DIRECTORY})
  set(CMAKE_LIBRARY_OUTPUT_DIRECTORY
      ${PROJECT_BINARY_DIR}/bin CACHE PATH "Single Directory for all Libraries"
  )
  set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY
      ${PROJECT_BINARY_DIR}/bin CACHE PATH "Single Directory for all static libraries."
  )
endif()

#------------------------------------------------------------------------------
# Disallow in-source build
#------------------------------------------------------------------------------
if("${HERMES_VFD_SOURCE_DIR}" STREQUAL "${HERMES_VFD_BINARY_DIR}")
  message(FATAL_ERROR
    "HERMES VFD requires an out of source Build. "
    "Please create a separate binary directory and run CMake there.")
endif()

#------------------------------------------------------------------------------
# Set a default build type if none was specified
#------------------------------------------------------------------------------
set(CMAKE_POSITION_INDEPENDENT_CODE ON)
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  message(STATUS "Setting build type to 'RelWithDebInfo' as none was specified.")
  set(CMAKE_BUILD_TYPE RelWithDebInfo CACHE STRING "Choose the type of build." FORCE)
  # Set the possible values of build type for cmake-gui
  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release"
    "MinSizeRel" "RelWithDebInfo")
endif()

#-----------------------------------------------------------------------------
# Targets built within this project are exported at Install time for use
# by other projects.
#-----------------------------------------------------------------------------
if(NOT HERMES_VFD_EXPORTED_TARGETS)
  set(HERMES_VFD_EXPORTED_TARGETS "${HERMES_VFD_PACKAGE}-targets")
endif()

#------------------------------------------------------------------------------
# Build options
#------------------------------------------------------------------------------
option(BUILD_SHARED_LIBS "Build with shared libraries." ON)
if(BUILD_SHARED_LIBS)
  set(HERMES_VFD_BUILD_SHARED_LIBS 1)
  set(HERMES_VFD_LIBTYPE SHARED)
else()
  set(HERMES_VFD_BUILD_SHARED_LIBS 0)
  set(HERMES_VFD_LIBTYPE STATIC)
endif()

#-------------------------------------------------------------------------------
function(hermes_vfd_set_lib_options libtarget libname libtype)
  if(${libtype} MATCHES "SHARED")
    set(LIB_RELEASE_NAME "${libname}")
    set(LIB_DEBUG_NAME "${libname}_debug")
    set(LIB_VERSION ${HERMES_VFD_PACKAGE_VERSION})
  else()
    # if the generator supports configuration types or if the CMAKE_BUILD_TYPE has a value
    if(CMAKE_CONFIGURATION_TYPES OR CMAKE_BUILD_TYPE)
      set(LIB_RELEASE_NAME "${libname}")
      set(LIB_DEBUG_NAME "${libname}_debug")
    else()
      set(LIB_RELEASE_NAME "lib${libname}")
      set(LIB_DEBUG_NAME "lib${libname}_debug")
    endif()
  endif()

  set_target_properties(${libtarget}
      PROPERTIES
      DEBUG_OUTPUT_NAME          ${LIB_DEBUG_NAME}
      RELEASE_OUTPUT_NAME        ${LIB_RELEASE_NAME}
      MINSIZEREL_OUTPUT_NAME     ${LIB_RELEASE_NAME}
      RELWITHDEBINFO_OUTPUT_NAME ${LIB_RELEASE_NAME}
      VERSION                    ${LIB_VERSION}
      SOVERSION                  ${LIB_VERSION}
  )

  #----- Use MSVC Naming conventions for Shared Libraries
  if(MINGW AND ${libtype} MATCHES "SHARED")
    set_target_properties(${libtarget}
      PROPERTIES
      IMPORT_SUFFIX ".lib"
      IMPORT_PREFIX ""
      PREFIX ""
    )
  endif()
endfunction()

#-----------------------------------------------------------------------------
# Source
#-----------------------------------------------------------------------------
add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/src)

#-----------------------------------------------------------------------------
# Configure the config.cmake file for the build directory
#-----------------------------------------------------------------------------
configure_file(
 ${HERMES_VFD_SOURCE_DIR}/CMake/hermes_vfd-config.cmake.build.in
 ${HERMES_VFD_BINARY_DIR}/hermes_vfd-config.cmake @ONLY
)

#-----------------------------------------------------------------------------
# Configure the config.cmake file for the install directory
#-----------------------------------------------------------------------------
configure_file(
 ${HERMES_VFD_SOURCE_DIR}/CMake/hermes_vfd-config.cmake.install.in
 ${HERMES_VFD_BINARY_DIR}/CMakeFiles/hermes_vfd-config.cmake @ONLY
)

install(
  FILES
    ${HERMES_VFD_BINARY_DIR}/CMakeFiles/hermes_vfd-config.cmake
  DESTINATION
    ${HERMES_VFD_INSTALL_DATA_DIR}/cmake/hermes_vfd
)

#-----------------------------------------------------------------------------
# Configure the hermes-config-version .cmake file for the install directory
#-----------------------------------------------------------------------------
configure_file(
  ${HERMES_VFD_SOURCE_DIR}/CMake/hermes_vfd-config-version.cmake.in
  ${HERMES_VFD_BINARY_DIR}/CMakeFiles/hermes_vfd-config-version.cmake @ONLY
)

install(
  FILES
    ${HERMES_VFD_BINARY_DIR}/CMakeFiles/hermes_vfd-config-version.cmake
  DESTINATION
    ${HERMES_VFD_INSTALL_DATA_DIR}/cmake/hermes_vfd
)
