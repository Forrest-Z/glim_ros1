cmake_minimum_required(VERSION 3.16)
project(glim_ros VERSION 1.0.6 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_CURRENT_LIST_DIR}/cmake")

if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  set(CMAKE_BUILD_TYPE "RelWithDebInfo" CACHE STRING "Choose the type of build." FORCE)
  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif()

option(BUILD_WITH_CUDA "Build with GPU support" OFF)
option(BUILD_WITH_VIEWER "Build with visualizer" ON)

find_package(catkin REQUIRED COMPONENTS roscpp cv_bridge image_transport tf2_ros roslib rosbag sensor_msgs glim)

# find_package(glim REQUIRED)
find_package(Eigen3 REQUIRED)
find_package(OpenCV REQUIRED core)
find_package(GTSAM REQUIRED)
find_package(spdlog REQUIRED)

###################################
## catkin specific configuration ##
###################################
catkin_package(
#  INCLUDE_DIRS include
#  LIBRARIES glim_ros
#  CATKIN_DEPENDS glim
#  DEPENDS glim
)

###########
## Build ##
###########

if(BUILD_WITH_VIEWER)
  add_definitions(-DBUILD_WITH_VIEWER)
endif()

if(BUILD_WITH_CUDA)
  add_definitions(-DBUILD_GTSAM_POINTS_GPU)
endif()

# if glim::glim does not exist
# if (NOT TARGET glim::glim)
#   message(WARNING "glim::glim does not exist, creating it")
#   add_library(glim::glim INTERFACE IMPORTED GLOBAL)
#   set_target_properties(glim::glim PROPERTIES
#     INTERFACE_INCLUDE_DIRECTORIES "${glim_INCLUDE_DIRS}"
#     INTERFACE_LINK_LIBRARIES "${glim_LIBRARIES}"
#   )

#   add_library(glim::interactive_viewer INTERFACE IMPORTED GLOBAL)
#   set_target_properties(glim::interactive_viewer PROPERTIES
#     INTERFACE_INCLUDE_DIRECTORIES "${glim_INCLUDE_DIRS}"
#     INTERFACE_LINK_LIBRARIES interactive_viewer
#   )
# endif()


add_library(glim_ros
  src/glim_ros/glim_ros.cpp
)
target_include_directories(glim_ros PUBLIC
  include
  ${catkin_INCLUDE_DIRS}
  ${EIGEN3_INCLUDE_DIR}
)
target_link_libraries(glim_ros
  # glim::glim
  ${catkin_LIBRARIES}
)

add_library(rviz_viewer SHARED
  src/glim_ros/rviz_viewer.cpp
)
target_include_directories(rviz_viewer PUBLIC
  include
  ${catkin_INCLUDE_DIRS}
  ${EIGEN3_INCLUDE_DIR}
)
target_link_libraries(rviz_viewer
  ${catkin_LIBRARIES}
  glim_ros
)

add_executable(glim_rosnode
  src/glim_rosnode.cpp
)
target_include_directories(glim_rosnode PUBLIC
  include
  ${catkin_INCLUDE_DIRS}
  ${EIGEN3_INCLUDE_DIR}
)
target_link_libraries(glim_rosnode
  ${catkin_LIBRARIES}
  glim_ros
)

add_executable(glim_rosbag
  src/glim_rosbag.cpp
)
target_include_directories(glim_rosbag PUBLIC
  include
  ${catkin_INCLUDE_DIRS}
  ${EIGEN3_INCLUDE_DIR}
)
target_link_libraries(glim_rosbag
  ${catkin_LIBRARIES}
  glim_ros
)


if(BUILD_WITH_VIEWER)
  add_executable(offline_viewer
    src/offline_viewer.cpp
  )
  target_include_directories(offline_viewer PUBLIC
    include
    ${catkin_INCLUDE_DIRS}
    ${EIGEN3_INCLUDE_DIR}
  )
  target_link_libraries(offline_viewer
    ${catkin_LIBRARIES}
    glim_ros
  )
endif()

#############
## Install ##
#############
install(DIRECTORY include/ DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION})

install(TARGETS glim_ros rviz_viewer glim_rosnode glim_rosbag offline_viewer
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)
