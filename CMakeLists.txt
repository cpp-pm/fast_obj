cmake_minimum_required(VERSION 3.0)

include("cmake/HunterGate.cmake")
HunterGate(
    URL "https://github.com/cpp-pm/hunter/archive/v0.23.273.tar.gz"
    SHA1 "43ffd06cf0352babdbe28d333adb15379ab59681"
)

project(fast_obj LANGUAGES C CXX VERSION 1.0)

# Introduce variables:
# * CMAKE_INSTALL_LIBDIR
# * CMAKE_INSTALL_BINDIR
# * CMAKE_INSTALL_INCLUDEDIR
include(GNUInstallDirs)

option(BUILD_AS_LIBRARY "Build library" FALSE)
option(BUILD_SHARED_LIBS "Build shared libraries" FALSE)
option(BUILD_TESTS "Build tests" FALSE)

if(MSVC AND BUILD_SHARED_LIBS)
  set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS TRUE)
endif()

set(FAST_OBJ_SRC_LIST fast_obj.c)
set(FAST_OBJ_HDR_LIST fast_obj.h)
if(BUILD_AS_LIBRARY)
  if(BUILD_SHARED_LIBS)
    add_library(fast_obj SHARED ${FAST_OBJ_SRC_LIST} ${FAST_OBJ_HDR_LIST})
  else()
    add_library(fast_obj STATIC ${FAST_OBJ_SRC_LIST} ${FAST_OBJ_HDR_LIST})
  endif()
else()
  message("making interface library")
  add_library(fast_obj INTERFACE)
endif()
target_include_directories(fast_obj INTERFACE
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
  $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}>
)

if(BUILD_TESTS)
  hunter_add_package(tinyobjloader)
  find_package(tinyobjloader CONFIG REQUIRED)

  set(TEST_SRC_LIST test/test.cpp)
  add_executable(test ${TEST_SRC_LIST})
  target_link_libraries(test PRIVATE fast_obj tinyobjloader::tinyobjloader)
endif()

set(GENERATED_DIR "${CMAKE_CURRENT_BINARY_DIR}/generated")
set(VERSION_CONFIG "${GENERATED_DIR}/${PROJECT_NAME}ConfigVersion.cmake")
set(PROJECT_CONFIG "${GENERATED_DIR}/${PROJECT_NAME}Config.cmake")

set(CONFIG_INSTALL_DIR "${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}")

set(TARGETS_EXPORT_NAME "${PROJECT_NAME}Targets")
set(NAMESPACE "${PROJECT_NAME}::")

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    "${VERSION_CONFIG}" COMPATIBILITY ExactVersion
)

configure_package_config_file(
    "cmake/Config.cmake.in"
    "${PROJECT_CONFIG}"
    INSTALL_DESTINATION "${CONFIG_INSTALL_DIR}"
)

install(
    TARGETS fast_obj
    EXPORT "${TARGETS_EXPORT_NAME}"
    INCLUDES DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}"
    RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}"
    LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}"
    ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}"
)

install(
    FILES ${FAST_OBJ_HDR_LIST}
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}
)

install(
    FILES "${PROJECT_CONFIG}" "${VERSION_CONFIG}"
    DESTINATION "${CONFIG_INSTALL_DIR}"
)

install(
    EXPORT "${TARGETS_EXPORT_NAME}"
    NAMESPACE "${NAMESPACE}"
    DESTINATION "${CONFIG_INSTALL_DIR}"
)
