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

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++17")

add_compile_options(-Wall)

#========================
# Dependencies Setup
#========================

find_package(rclcpp REQUIRED)

add_definitions(-DROS2_FOUND)
include_directories(${rclcpp_INCLUDE_DIRS})
set(CMAKE_CXX_STANDARD 14)

find_package(ament_cmake REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(std_msgs REQUIRED)  
find_package(rclcpp_action REQUIRED)
find_package(rosidl_typesupport_c REQUIRED) 
find_package(rosidl_default_generators REQUIRED)
find_package(builtin_interfaces REQUIRED)
find_package(std_msgs REQUIRED)
find_package(rclcpp_components REQUIRED)

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

add_library(hesai_ros_driver_component SHARED
  node/hesai_ros_driver_node.cc
  src/driver/HesaiLidar_SDK_2.0/libhesai/Lidar/lidar.cc
  src/driver/HesaiLidar_SDK_2.0/libhesai/UdpParser/udp_parser.cc
  src/driver/HesaiLidar_SDK_2.0/libhesai/PtcClient/src/ptc_client.cc
  src/driver/HesaiLidar_SDK_2.0/libhesai/PtcClient/src/tcp_client.cc
  src/driver/HesaiLidar_SDK_2.0/libhesai/PtcClient/src/tcp_ssl_client.cc
  src/driver/HesaiLidar_SDK_2.0/libhesai/PtcParser/src/general_ptc_parser.cc
  src/driver/HesaiLidar_SDK_2.0/libhesai/PtcParser/src/ptc_1_0_parser.cc
  src/driver/HesaiLidar_SDK_2.0/libhesai/PtcParser/src/ptc_2_0_parser.cc
  src/driver/HesaiLidar_SDK_2.0/libhesai/PtcParser/ptc_parser.cc
  src/driver/HesaiLidar_SDK_2.0/libhesai/Source/src/source.cc
  src/driver/HesaiLidar_SDK_2.0/libhesai/Source/src/socket_source.cc
  src/driver/HesaiLidar_SDK_2.0/libhesai/Source/src/pcap_source.cc
  src/driver/HesaiLidar_SDK_2.0/libhesai/Source/src/serial_source.cc
  src/driver/HesaiLidar_SDK_2.0/libhesai/Source/src/pcap_saver.cc
  src/driver/HesaiLidar_SDK_2.0/libhesai/Source/src/tcp_source.cc
  src/driver/HesaiLidar_SDK_2.0/libhesai/SerialClient/src/serial_client.cc
  src/driver/HesaiLidar_SDK_2.0/libhesai/Logger/src/logger.cc
  src/driver/HesaiLidar_SDK_2.0/libhesai/Common/src/plat_utils.cc
  src/driver/HesaiLidar_SDK_2.0/libhesai/Common/src/auto_tick_count.cc
)
target_link_libraries(hesai_ros_driver_component                   
  ${YAML_CPP_LIBRARIES}
)  
rclcpp_components_register_node(
  hesai_ros_driver_component
  PLUGIN "SourceDriver"
  EXECUTABLE hesai_ros_driver_node
)          

target_include_directories(hesai_ros_driver_component PRIVATE
  src/driver/HesaiLidar_SDK_2.0/
  src/driver/HesaiLidar_SDK_2.0/libhesai
  src/driver/HesaiLidar_SDK_2.0/libhesai/Lidar
  src/driver/HesaiLidar_SDK_2.0/libhesai/UdpParser
  src/driver/HesaiLidar_SDK_2.0/libhesai/UdpParser/include
  src/driver/HesaiLidar_SDK_2.0/libhesai/UdpParser/src
  src/driver/HesaiLidar_SDK_2.0/libhesai/UdpProtocol
  src/driver/HesaiLidar_SDK_2.0/libhesai/Source/include
  src/driver/HesaiLidar_SDK_2.0/libhesai/Container/include
  src/driver/HesaiLidar_SDK_2.0/libhesai/Container/src
  src/driver/HesaiLidar_SDK_2.0/libhesai/UdpParserGpu
  src/driver/HesaiLidar_SDK_2.0/libhesai/UdpParserGpu/include
  src/driver/HesaiLidar_SDK_2.0/libhesai/UdpParserGpu/src
  src/driver/HesaiLidar_SDK_2.0/libhesai/PtcClient/include
  src/driver/HesaiLidar_SDK_2.0/libhesai/PtcParser/include
  src/driver/HesaiLidar_SDK_2.0/libhesai/PtcParser
  src/driver/HesaiLidar_SDK_2.0/libhesai/Logger/include
  src/driver/HesaiLidar_SDK_2.0/libhesai/include
  src/driver/HesaiLidar_SDK_2.0/libhesai/SerialClient/include
  src/driver/HesaiLidar_SDK_2.0/libhesai/Common/include
  src/driver/HesaiLidar_SDK_2.0/driver
  src/driver/HesaiLidar_SDK_2.0/algorithm
  src/manager
  src/msg/ros_msg
  src/msg/rs_msg
  src/utility
  ${CMAKE_CURRENT_BINARY_DIR}
)

find_package(ament_index_cpp REQUIRED)
ament_target_dependencies(hesai_ros_driver_component
  ament_index_cpp
  rcl_interfaces
  rclcpp
  rclcpp_components
  rcutils
  std_msgs
  sensor_msgs
)
rosidl_get_typesupport_target(cpp_typesupport_target "${PROJECT_NAME}" "rosidl_typesupport_cpp")
target_link_libraries(hesai_ros_driver_component "${cpp_typesupport_target}")

ament_export_targets(export_hesai_ros_driver_component)
install(TARGETS hesai_ros_driver_component ${LIB_TARGETS}
        EXPORT export_hesai_ros_driver_component
        ARCHIVE DESTINATION lib
        LIBRARY DESTINATION lib
        RUNTIME DESTINATION bin
)

install(TARGETS
  hesai_ros_driver_node
  DESTINATION lib/${PROJECT_NAME})
install(TARGETS
        ${LIB_TARGETS}
        DESTINATION lib)

install(DIRECTORY
        launch
        rviz
        DESTINATION share/${PROJECT_NAME})

ament_package()


