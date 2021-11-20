# CMakeLists files in this project can
# refer to the root source directory of the project as ${HDF5_HERMES_SOURCE_DIR} and
# to the root binary directory of the project as ${HDF5_HERMES_BINARY_DIR}.
cmake_minimum_required (VERSION 3.10)

if(CMAKE_SIZEOF_VOID_P LESS 8)
  message(FATAL_ERROR "Hermes currently requires a 64-bit processor")
endif()

project(HDF5_HERMES_VFD)

include_directories(${PROJECT_BINARY_DIR})

if(NOT CMAKE_INSTALL_RPATH_USE_LINK_PATH)
  set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)
endif()

#------------------------------------------------------------------------------
# Set module path
#------------------------------------------------------------------------------
set(HDF5_HERMES_VFD_CMAKE_MODULE_PATH "${HDF5_HERMES_VFD_SOURCE_DIR}/CMake")
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${HDF5_HERMES_VFD_CMAKE_MODULE_PATH})

#------------------------------------------------------------------------------
# Version information
#------------------------------------------------------------------------------
set(HDF5_HERMES_VFD_VERSION_MAJOR "0")
set(HDF5_HERMES_VFD_VERSION_MINOR "1")
set(HDF5_HERMES_VFD_VERSION_PATCH "0")
set(HDF5_HERMES_VFD_PACKAGE "hdf5_hermes_vfd")
set(HDF5_HERMES_VFD_PACKAGE_NAME "HDF5_HERMES_VFD")
set(HDF5_HERMES_VFD_PACKAGE_VERSION "${HDF5_HERMES_VFD_VERSION_MAJOR}.${HDF5_HERMES_VFD_VERSION_MINOR}.${HDF5_HERMES_VFD_VERSION_PATCH}")
set(HDF5_HERMES_VFD_PACKAGE_VERSION_MAJOR "${HDF5_HERMES_VFD_VERSION_MAJOR}.${HDF5_HERMES_VFD_VERSION_MINOR}")
set(HDF5_HERMES_VFD_PACKAGE_VERSION_MINOR "${HDF5_HERMES_VFD_VERSION_PATCH}")
set(HDF5_HERMES_VFD_PACKAGE_STRING "${HDF5_HERMES_VFD_PACKAGE_NAME} ${HDF5_HERMES_VFD_PACKAGE_VERSION}")
set(HDF5_HERMES_VFD_PACKAGE_TARNAME "${HDF5_HERMES_VFD_PACKAGE}")

#------------------------------------------------------------------------------
# Setup install and output Directories
#------------------------------------------------------------------------------
if(NOT HDF5_HERMES_VFD_INSTALL_BIN_DIR)
  set(HDF5_HERMES_VFD_INSTALL_BIN_DIR ${CMAKE_INSTALL_PREFIX}/bin)
endif()
if(NOT HDF5_HERMES_VFD_INSTALL_LIB_DIR)
  set(HDF5_HERMES_VFD_INSTALL_LIB_DIR ${CMAKE_INSTALL_PREFIX}/lib)
endif()
if(NOT HDF5_HERMES_VFD_INSTALL_INCLUDE_DIR)
  set(HDF5_HERMES_VFD_INSTALL_INCLUDE_DIR ${CMAKE_INSTALL_PREFIX}/include)
endif()
if(NOT HDF5_HERMES_VFD_INSTALL_DATA_DIR)
  set(HDF5_HERMES_VFD_INSTALL_DATA_DIR ${CMAKE_INSTALL_PREFIX}/share)
endif()

#------------------------------------------------------------------------------
if(NOT HDF5_HERMES_VFD_EXTERNALLY_CONFIGURED)
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
if("${HDF5_HERMES_VFD_SOURCE_DIR}" STREQUAL "${HDF5_HERMES_VFD_BINARY_DIR}")
  message(FATAL_ERROR
    "HDF5_HERMES VFD requires an out of source Build. "
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
if(NOT HDF5_HERMES_VFD_EXPORTED_TARGETS)
  set(HDF5_HERMES_VFD_EXPORTED_TARGETS "${HDF5_HERMES_VFD_PACKAGE}-targets")
endif()

#------------------------------------------------------------------------------
# Build options
#------------------------------------------------------------------------------
option(BUILD_SHARED_LIBS "Build with shared libraries." ON)
if(BUILD_SHARED_LIBS)
  set(HDF5_HERMES_VFD_BUILD_SHARED_LIBS 1)
  set(HDF5_HERMES_VFD_LIBTYPE SHARED)
else()
  set(HDF5_HERMES_VFD_BUILD_SHARED_LIBS 0)
  set(HDF5_HERMES_VFD_LIBTYPE STATIC)
endif()

#-------------------------------------------------------------------------------
function(hdf5_hermes_vfd_set_lib_options libtarget libname libtype)
  if(${libtype} MATCHES "SHARED")
    set(LIB_RELEASE_NAME "${libname}")
    set(LIB_DEBUG_NAME "${libname}_debug")
    set(LIB_VERSION ${HDF5_HERMES_VFD_PACKAGE_VERSION})
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

#------------------------------------------------------------------------------
# External dependencies
#------------------------------------------------------------------------------
#HDF5
find_package(HDF5 1.13.0 NO_MODULE NAMES hdf5 COMPONENTS C shared)
if(HDF5_FOUND)
  set(HDF5_C_SHARED_LIBRARY hdf5-shared)
#  if(NOT TARGET ${HDF5_C_SHARED_LIBRARY})
#      message(FATAL_ERROR "Could not find hdf5 shared target, please make "
#"sure that HDF5 has ben compiled with shared libraries enabled.")
#    endif()
  set(HDF5_HERMES_VFD_EXT_INCLUDE_DEPENDENCIES
      ${HDF5_HERMES_VFD_EXT_INCLUDE_DEPENDENCIES}
      ${HDF5_INCLUDE_DIR}
     )
  set(HDF5_HERMES_VFD_EXT_PKG_DEPENDENCIES
      ${HDF5_HERMES_VFD_EXT_PKG_DEPENDENCIES}
      ${HDF5_C_SHARED_LIBRARY}
     )
else()
# Allow for HDF5 autotools builds
  find_package(HDF5 1.13.0 MODULE REQUIRED)
  if(HDF5_FOUND)
    set(HDF5_HERMES_VFD_EXT_INCLUDE_DEPENDENCIES
        ${HDF5_HERMES_VFD_EXT_INCLUDE_DEPENDENCIES}
        ${HDF5_INCLUDE_DIRS}
       )
    set(HDF5_HERMES_VFD_EXT_LIB_DEPENDENCIES
        ${HDF5_HERMES_VFD`_EXT_LIB_DEPENDENCIES}
        ${HDF5_LIBRARIES}
       )
  else()
    message(FATAL_ERROR "Could not find HDF5, please check HDF5_DIR.")
  endif()
endif()

#Hermes
find_package(HERMES REQUIRED CONFIG)
if(HERMES_FOUND)
  set(HDF5_HERMES_VFD_EXT_INCLUDE_DEPENDENCIES
    ${HERMES_VFD_EXT_INCLUDE_DEPENDENCIES}
    ${HERMES_INCLUDE_DIR}
  )
  set(HDF5_HERMES_VFD_EXT_LIB_DEPENDENCIES
    ${HERMES_VFD_EXT_LIB_DEPENDENCIES}
    hermes_wrapper
  )
else()
  message(FATAL_ERROR "Could not find hermes.")
endif()

#MPI
option(HERMES_VFD_ENABLE_MPI "Enable MPI." ON)
if(HERMES_VFD_ENABLE_MPI)
  find_package(MPI REQUIRED)
  if(MPI_FOUND)
    message(STATUS "found mpi.h at ${MPI_INCLUDE_PATH}")
    set(HDF5_HERMES_VFD_EXT_INCLUDE_DEPENDENCIES
      ${HDF5_HERMES_VFD_EXT_INCLUDE_DEPENDENCIES}
      ${MPI_INCLUDE_PATH}
    )
    set(HDF5_HERMES_VFD_EXT_LIB_DEPENDENCIES
      ${HDF5_HERMES_VFD_EXT_LIB_DEPENDENCIES}
      ${MPI_LIBRARIES}
    )
  else()
    message(FATAL_ERROR "Could not find MPI.")
  endif()
endif()

#-----------------------------------------------------------------------------
# Source
#-----------------------------------------------------------------------------
add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/src)

#-----------------------------------------------------------------------------
# Testing
#-----------------------------------------------------------------------------
option(BUILD_TESTING "Build testing." ON)
if(NOT HDF5_HERMES_VFD_EXTERNALLY_CONFIGURED AND BUILD_TESTING)
  enable_testing()
  include(CTest)
  add_subdirectory(test)
endif()

#-----------------------------------------------------------------------------
# Configure the config.cmake file for the build directory
#-----------------------------------------------------------------------------
configure_file(
 ${HDF5_HERMES_VFD_SOURCE_DIR}/CMake/hdf5_hermes_vfd-config.cmake.build.in
 ${HDF5_HERMES_VFD_BINARY_DIR}/hdf5_hermes_vfd-config.cmake @ONLY
)

#-----------------------------------------------------------------------------
# Configure the config.cmake file for the install directory
#-----------------------------------------------------------------------------
configure_file(
 ${HDF5_HERMES_VFD_SOURCE_DIR}/CMake/hdf5_hermes_vfd-config.cmake.install.in
 ${HDF5_HERMES_VFD_BINARY_DIR}/CMakeFiles/hdf5_hermes_vfd-config.cmake @ONLY
)

install(
  FILES
    ${HDF5_HERMES_VFD_BINARY_DIR}/CMakeFiles/hdf5_hermes_vfd-config.cmake
  DESTINATION
    ${HDF5_HERMES_VFD_INSTALL_DATA_DIR}/cmake/hdf5_hermes_vfd
)

#-----------------------------------------------------------------------------
# Configure the hermes-config-version .cmake file for the install directory
#-----------------------------------------------------------------------------
configure_file(
  ${HDF5_HERMES_VFD_SOURCE_DIR}/CMake/hdf5_hermes_vfd-config-version.cmake.in
  ${HDF5_HERMES_VFD_BINARY_DIR}/CMakeFiles/hdf5_hermes_vfd-config-version.cmake @ONLY
)

install(
  FILES
    ${HDF5_HERMES_VFD_BINARY_DIR}/CMakeFiles/hdf5_hermes_vfd-config-version.cmake
  DESTINATION
    ${HDF5_HERMES_VFD_INSTALL_DATA_DIR}/cmake/hdf5_hermes_vfd
)
