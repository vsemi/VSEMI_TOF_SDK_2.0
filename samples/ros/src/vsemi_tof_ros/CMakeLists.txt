cmake_minimum_required(VERSION 3.5.0)
project(vsemi_tof_ros)

EXECUTE_PROCESS( COMMAND uname -m COMMAND tr -d '\n' OUTPUT_VARIABLE ARCHITECTURE )

message( STATUS "Architecture: ${ARCHITECTURE}" )

add_compile_options(-std=c++14)

find_package(catkin REQUIRED COMPONENTS
    dynamic_reconfigure
    roscpp
    sensor_msgs
    std_msgs
    pcl_conversions
    pcl_ros
    tf
)

find_package(OpenCV REQUIRED)

message("OpenCV version: ${OpenCV_VERSION}")

################################################
## Declare ROS dynamic reconfigure parameters ##
################################################

generate_dynamic_reconfigure_options(cfg/vsemi_tof_ros.cfg)

###################################
## catkin specific configuration ##
###################################

catkin_package(
  INCLUDE_DIRS include
  LIBRARIES vsemi_tof_cam
  CATKIN_DEPENDS roscpp dynamic_reconfigure message_runtime sensor_msgs std_msgs
)

###########
## Build ##
###########

include_directories(
  include
  ../../../../driver/include  
  ${catkin_INCLUDE_DIRS}
)

link_directories(
  ../../../../driver/lib/${ARCHITECTURE}
)

add_executable(tof_cam_node
  src/tof_cam_node.cpp
)

######################################################################
# make sure configure headers are built before any node using them   #
######################################################################

add_dependencies(tof_cam_node ${PROJECT_NAME}_gencfg)

add_dependencies(tof_cam_node ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})

target_link_libraries(tof_cam_node ${catkin_LIBRARIES})

target_link_libraries(tof_cam_node ${OpenCV_LIBS})

target_link_libraries(tof_cam_node vsemi_tof_cam)

#############
## Install ##
#############

install(TARGETS tof_cam_node
   ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
   LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
   RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

install(DIRECTORY launch  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION})

