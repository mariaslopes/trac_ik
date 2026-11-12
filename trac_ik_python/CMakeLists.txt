cmake_minimum_required(VERSION 3.16)
project(trac_ik_python)

# ROS 2 port: SWIG Python wrapper around trac_ik_lib, built with ament_cmake.

find_package(ament_cmake REQUIRED)
find_package(ament_cmake_python REQUIRED)
find_package(trac_ik_lib REQUIRED)
find_package(kdl_parser REQUIRED)
find_package(urdf REQUIRED)
find_package(rclcpp REQUIRED)
find_package(SWIG 4 REQUIRED)
find_package(Python3 REQUIRED COMPONENTS Interpreter Development.Module)
include(UseSWIG)

set_property(SOURCE swig/trac_ik_wrap.i PROPERTY CPLUSPLUS ON)
swig_add_library(trac_ik_wrap
  TYPE MODULE
  LANGUAGE python
  OUTPUT_DIR ${CMAKE_CURRENT_BINARY_DIR}/trac_ik_python
  SOURCES swig/trac_ik_wrap.i)
target_compile_features(trac_ik_wrap PRIVATE cxx_std_17)
target_compile_options(trac_ik_wrap PRIVATE -Wno-deprecated)
# DT_RPATH (not RUNPATH) so libtrac_ik.so's own dependencies resolve as well
target_link_options(trac_ik_wrap PRIVATE -Wl,--disable-new-dtags)
target_link_libraries(trac_ik_wrap PRIVATE
  trac_ik_lib::trac_ik kdl_parser::kdl_parser urdf::urdf rclcpp::rclcpp Python3::Module)
set_target_properties(trac_ik_wrap PROPERTIES
  SWIG_USE_TARGET_INCLUDE_DIRECTORIES TRUE
  # find libtrac_ik and the ROS libraries without sourcing a setup file
  INSTALL_RPATH_USE_LINK_PATH TRUE)

ament_python_install_package(${PROJECT_NAME} PACKAGE_DIR src/${PROJECT_NAME})
install(TARGETS trac_ik_wrap DESTINATION ${PYTHON_INSTALL_DIR}/${PROJECT_NAME})
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/trac_ik_python/trac_ik_wrap.py
  DESTINATION ${PYTHON_INSTALL_DIR}/${PROJECT_NAME})

ament_package()
