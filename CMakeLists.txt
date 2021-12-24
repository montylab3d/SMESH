cmake_minimum_required(VERSION 3.3)
project(SMESH VERSION 9.7.0.1 LANGUAGES C CXX)

# --------------------------------------------------------------------------- #
# OPTIONS
# --------------------------------------------------------------------------- #
set(CMAKE_INSTALL_PREFIX "${CMAKE_SOURCE_DIR}/install" CACHE PATH "Installation directory")
if(UNIX)
	include(GNUInstallDirs)
	mark_as_advanced(CLEAR CMAKE_INSTALL_BINDIR)
	mark_as_advanced(CLEAR CMAKE_INSTALL_LIBDIR)
else()
	set(CMAKE_INSTALL_LIBDIR lib CACHE PATH "Output directory for libraries")
	set(CMAKE_INSTALL_BINDIR bin CACHE PATH "Output directory for binaries")
endif()


# --------------------------------------------------------------------------- #
# SETTINGS
# --------------------------------------------------------------------------- #
set(SMESH_VERSION_MAJOR 9)
set(SMESH_VERSION_MINOR 7)
set(SMESH_VERSION_PATCH 0)
set(SMESH_VERSION_TWEAK 1)

# Build shared libraries
set(BUILD_SHARED_LIBS TRUE)

# Force C++ 17
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
# Do *not* force extensions on or off-- this will cause ABI problems with gnu++17

# Warning and errors
if(MSVC)
    add_compile_options(/W1)
    add_compile_options(/wd4715 /wd4091 /wd4503 /wd4805 /wd4267 /wd4477 /wd4138 /wd4273 /wd4311 /wd4806)
else()
    add_compile_options(-Wno-deprecated -Wno-sign-compare -Wno-unused-variable -Wno-unused-but-set-variable)
endif()

# Definitions
if(UNIX)
  # Same settings are used for both MacOSX and Unix/Linux
  add_definitions(-DHAVE_LIMITS_H -DCSFDB -DLIN -DOCC_CONVERT_SIGNALS)
else(UNIX)
  if(WIN32)
      if(MSVC)
          add_definitions(-DWNT -DWIN32 -D_WINDOWS -DCSFDB -DUSE_CLOCK -DMSDOS -DNO_ONEXIT -DNO_My_ctype -DNO_ISATTY -DNO_FPINIT)
      else(MSVC)
          add_definitions(-DWNT -DWIN32 -D_WINDOWS -DCSFDB)
      endif(MSVC)
  else(WIN32)
      message(FATAL_ERROR "Unknown platform")
  endif(WIN32)
endif(UNIX)

if(CMAKE_SIZEOF_VOID_P STREQUAL 8)
    add_definitions(-D_OCC64)
    # add_definitions(-DSALOME_USE_64BIT_IDS)
endif(CMAKE_SIZEOF_VOID_P STREQUAL 8)

set(CMAKE_VERBOSE_MAKEFILE ON)

# --------------------------------------------------------------------------- #
# OpenCASCADE
# --------------------------------------------------------------------------- #
message(STATUS "Searching for OpenCASCADE...")
find_package(OpenCASCADE REQUIRED)

include_directories(${OpenCASCADE_INCLUDE_DIR})
link_directories(${OpenCASCADE_LIBRARY_DIR})

# --------------------------------------------------------------------------- #
# VTK
# --------------------------------------------------------------------------- #
message(STATUS "Searching for VTK...")
find_package(VTK REQUIRED COMPONENTS CommonCore CommonDataModel FiltersVerdict)


# --------------------------------------------------------------------------- #
# BOOST
# --------------------------------------------------------------------------- #
message(STATUS "Searching for Boost...")
find_package(Boost REQUIRED COMPONENTS filesystem thread serialization)

if (MSVC)
  # find the shared boost libs
  add_definitions(-DBOOST_ALL_DYN_LINK)
  # set postfix for debug libs
  if(NOT CMAKE_DEBUG_POSTFIX)
      set(CMAKE_DEBUG_POSTFIX d)
  endif()
endif()


# --------------------------------------------------------------------------- #
# PTHREAD
# --------------------------------------------------------------------------- #
if(UNIX)
  find_package(Threads REQUIRED)
else(UNIX)
  set(PTHREAD_INCLUDE_DIRS "" CACHE PATH "pthread include directory.")
  if(NOT EXISTS ${PTHREAD_INCLUDE_DIRS})
    message(FATAL_ERROR "pthread include directory is required.")
  endif()
  include_directories(${PTHREAD_INCLUDE_DIRS})

  set(PTHREAD_LIB_DIRS "" CACHE PATH "pthread library directory.")
  if(NOT EXISTS ${PTHREAD_LIB_DIRS})
    message(FATAL_ERROR "pthread library directory is required.")
  endif()
  link_directories(${PTHREAD_LIB_DIRS})
endif(UNIX)


# --------------------------------------------------------------------------- #
# NetGen
# --------------------------------------------------------------------------- #
message(STATUS "Searching for Netgen...")
find_package(Netgen REQUIRED)

include_directories(
  ${NETGEN_INCLUDE_DIR}
  ${NETGEN_INCLUDE_DIR}/core
  ${NETGEN_INCLUDE_DIR}/csg
  ${NETGEN_INCLUDE_DIR}/general
  ${NETGEN_INCLUDE_DIR}/geom2d
  ${NETGEN_INCLUDE_DIR}/gprim
  ${NETGEN_INCLUDE_DIR}/include
  ${NETGEN_INCLUDE_DIR}/interface
  ${NETGEN_INCLUDE_DIR}/linalg
  ${NETGEN_INCLUDE_DIR}/gprim
  ${NETGEN_INCLUDE_DIR}/meshing
  ${NETGEN_INCLUDE_DIR}/occ
  ${NETGEN_INCLUDE_DIR}/stlgeom
  ${NETGEN_INCLUDE_DIR}/visualization
  )
link_directories(${NETGEN_LIBRARY_DIR})

# --------------------------------------------------------------------------- #
# SMESH
# --------------------------------------------------------------------------- #

# Kernel
set(Kernel_SRC_DIR ${CMAKE_SOURCE_DIR}/kernel/src)

# Geom
set(Geom_SRC_DIR ${CMAKE_SOURCE_DIR}/geom/src)

add_subdirectory(kernel)
add_subdirectory(geom)
add_subdirectory(smesh)

get_directory_property(Kernel_LIBRARIES
  DIRECTORY ${CMAKE_SOURCE_DIR}/kernel
  DEFINITION Kernel_LIBRARIES
)

get_directory_property(Geom_LIBRARIES
  DIRECTORY ${CMAKE_SOURCE_DIR}/geom
  DEFINITION Geom_LIBRARIES
)

get_directory_property(SMESH_LIBRARIES
  DIRECTORY ${CMAKE_SOURCE_DIR}/smesh
  DEFINITION SMESH_LIBRARIES
)

# --------------------------------------------------------------------------- #
# Install
# --------------------------------------------------------------------------- #
install(TARGETS ${Kernel_LIBRARIES} ${Geom_LIBRARIES} ${SMESH_LIBRARIES} EXPORT SMESH-targets
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR})

if(UNIX)
	set(CMAKE_EXPORT_DIR ${CMAKE_INSTALL_LIBDIR}/cmake/SMESH)
else()
	set(CMAKE_EXPORT_DIR cmake)
endif()

install(EXPORT SMESH-targets DESTINATION ${CMAKE_EXPORT_DIR})


# --------------------------------------------------------------------------- #
# Configuration files
# --------------------------------------------------------------------------- #
include(CMakePackageConfigHelpers)
configure_file(${CMAKE_SOURCE_DIR}/cmake/SMESHConfig.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/SMESHConfig.cmake @ONLY)

write_basic_package_version_file(${CMAKE_CURRENT_BINARY_DIR}/SMESHConfigVersion.cmake
  VERSION ${SMESH_VERSION_MAJOR}.${SMESH_VERSION_MINOR}.${SMESH_VERSION_PATCH}
  COMPATIBILITY SameMinorVersion)

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/SMESHConfig.cmake
              ${CMAKE_CURRENT_BINARY_DIR}/SMESHConfigVersion.cmake
  DESTINATION ${CMAKE_EXPORT_DIR})
