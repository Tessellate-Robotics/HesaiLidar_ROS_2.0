cmake_minimum_required(VERSION 3.5)
cmake_policy(SET CMP0048 NEW)
project(hesai_ros_driver)

#=======================================
# Version information
#=======================================
set(VERSION_MAJOR 2)
set(VERSION_MINOR 0)
set(VERSION_TINY  11)
configure_file(
  "${CMAKE_CURRENT_SOURCE_DIR}/Version.h.in"
  "${CMAKE_CURRENT_BINARY_DIR}/Version.h"
)

set(WITH_PTCS_USE ON CACHE BOOL "Use Ptcs")

#=======================================
# Custom Point Type (XYZI, XYZIRT)
#=======================================
set(POINT_TYPE XYZI)

#========================
# Project details / setup
#========================
set(PROJECT_NAME hesai_ros_driver)

add_definitions(-DPROJECT_PATH="${PROJECT_SOURCE_DIR}")

if (CMAKE_BUILD_TYPE STREQUAL "")
  set(CMAKE_BUILD_TYPE Release)
  add_definitions(-O3)
endif()

if(($ENV{ROS_DISTRO} STREQUAL "rolling") OR
   ($ENV{ROS_DISTRO} STREQUAL "kilted") OR
   ($ENV{ROS_DISTRO} STREQUAL "jazzy") OR
   ($ENV{ROS_DISTRO} STREQUAL "humble"))
  set(CMAKE_CXX_STANDARD 17)
else()
  set(CMAKE_CXX_STANDARD 14)
endif()
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_compile_options(-Wall)

#========================
# Dependencies Setup
#========================

#ROS#
add_definitions(-DROS2_FOUND)

find_package(rclcpp REQUIRED)
find_package(ament_cmake REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(std_msgs REQUIRED)
find_package(rclcpp_action REQUIRED)
find_package(rosidl_default_generators REQUIRED)
find_package(builtin_interfaces REQUIRED)
find_package(rclcpp_components REQUIRED)
find_package(ament_index_cpp REQUIRED)
find_package(rcl_interfaces REQUIRED)
find_package(rcutils REQUIRED)

rosidl_generate_interfaces(${PROJECT_NAME}
  "msg/msg_ros2/UdpPacket.msg"
  "msg/msg_ros2/UdpFrame.msg"
  "msg/msg_ros2/Firetime.msg"
  "msg/msg_ros2/Ptp.msg"
  "msg/msg_ros2/LossPacket.msg"
  DEPENDENCIES builtin_interfaces std_msgs
)
ament_export_dependencies(rosidl_default_runtime)



#Others#
find_package(yaml-cpp REQUIRED)

#Include directory#
include_directories(${PROJECT_SOURCE_DIR}/src)

#Driver core#
set(DISENABLE_TEST_CC ON CACHE BOOL "DISENABLE_TEST_CC")
add_subdirectory(src/driver/HesaiLidar_SDK_2.0)

#========================
# Build Setup
#========================

find_package(Boost REQUIRED COMPONENTS thread)
include_directories(${Boost_INCLUDE_DIRS})
link_directories(${Boost_LIBRARY_DIRS})

set(LIB_TARGETS
    hesai_sdk_lib
    log_lib
    lidar_lib
    ptcClient_lib
    source_lib
    platutils_lib
    serialClient_lib
    udpParser_lib
    )

add_library(hesai_ros_driver_component SHARED
  node/hesai_ros_driver_node.cc
)
target_link_libraries(hesai_ros_driver_component PRIVATE
            ${YAML_CPP_LIBRARIES}
            ${Boost_LIBRARIES}
            ${LIB_TARGETS}
)

rclcpp_components_register_node(
  hesai_ros_driver_component
  PLUGIN "SourceDriver"
  EXECUTABLE hesai_ros_driver_node
)

target_include_directories(hesai_ros_driver_component PRIVATE
  src/manager
  src/msg/ros_msg
  src/msg/rs_msg
  src/utility
  ${CMAKE_CURRENT_BINARY_DIR}
)

rosidl_get_typesupport_target(cpp_typesupport_target "${PROJECT_NAME}" "rosidl_typesupport_cpp")
target_link_libraries(hesai_ros_driver_component PUBLIC
  ament_index_cpp::ament_index_cpp
  ${rcl_interfaces_TARGETS}
  rclcpp::rclcpp
  rclcpp_components::component
  ${rcutils_TARGETS}
  ${std_msgs_TARGETS}
  ${sensor_msgs_TARGETS}
  "${cpp_typesupport_target}"
)

install(TARGETS hesai_ros_driver_component
        ARCHIVE DESTINATION lib
        LIBRARY DESTINATION lib
        RUNTIME DESTINATION bin
)

install(TARGETS
  hesai_ros_driver_node
  DESTINATION lib/${PROJECT_NAME})

install(DIRECTORY
        launch
        rviz
        DESTINATION share/${PROJECT_NAME})

ament_package()
