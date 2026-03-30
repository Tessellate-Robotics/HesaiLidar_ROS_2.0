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
# set(FIND_CUDA ON CACHE BOOL "Find CUDA")

if(FIND_CUDA)
    find_package(CUDA )
endif()
if(${CUDA_FOUND})
  	project(CommonModules LANGUAGES CXX CUDA)
	set(CUDA_SOURCE_PROPERTY_FORMAT OBJ)
	set(CUDA_SEPARABLE_COMPILATION ON)
	include_directories(${CUDA_INCLUDE_DIRS})
	set(CUDA_PROPAGATE_HOST_FLAGS OFF)
	set(CUDA_NVCC_FLAGS -arch=sm_61;-O3;-G;-g;)#根据具体GPU性能更改算力参数
	link_directories($ENV{CUDA_PATH}/lib/x64)

  add_definitions(-DUSE_CUDA)
  set(FIND_CUDA ON CACHE BOOL "Use Cuda")
else(${CUDA_FOUND})
	MESSAGE(STATUS "cuda not found!")
endif(${CUDA_FOUND})

#=======================================
# Custom Point Type (XYZI, XYZIRT)
#=======================================
set(POINT_TYPE XYZI)

#=======================================
# Compile setup (ORIGINAL, CATKIN, COLCON)
#=======================================
message(=============================================================)
message("-- ROS_VERSION is $ENV{ROS_VERSION}")
message(=============================================================)


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
find_package(ament_cmake REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(std_msgs REQUIRED)
find_package(rclcpp_action REQUIRED)
find_package(rosidl_default_generators REQUIRED)
find_package(builtin_interfaces REQUIRED)
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
  src/manager/node_manager.cc
  )
target_link_libraries(hesai_ros_driver_component
            ${YAML_CPP_LIBRARIES}
            ${Boost_LIBRARIES}
            ${LIB_TARGETS}
)

rclcpp_components_register_node(
  hesai_ros_driver_component
  PLUGIN "SourceDriver"
  EXECUTABLE hesai_ros_driver_node
)

target_include_directories(hesai_ros_driver_node PRIVATE
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
