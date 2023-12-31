cmake_minimum_required(VERSION 2.8.3)
project(lattice_planner)

find_package(autoware_build_flags REQUIRED)

find_package(
  catkin REQUIRED COMPONENTS
    roscpp
    std_msgs
    pcl_ros
    pcl_conversions
    sensor_msgs
    tf
    gnss
    libwaypoint_follower
    vector_map
    waypoint_planner
    autoware_can_msgs
    vector_map
)

find_package(Armadillo REQUIRED)

OPTION(USE_OpenMP "Use OpenMP" ON)
IF (USE_OpenMP)
    FIND_PACKAGE(OpenMP)
    IF (OPENMP_FOUND)
        SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
        SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
    ENDIF ()
ENDIF ()


catkin_package(
  INCLUDE_DIRS include
  LIBRARIES libtraj_gen
  CATKIN_DEPENDS
    autoware_can_msgs
    gnss
    libwaypoint_follower
    pcl_conversions
    pcl_ros
    roscpp
    sensor_msgs
    std_msgs
    tf
    vector_map
    waypoint_planner
)

SET(CMAKE_CXX_FLAGS "-O2 -g -Wall ${CMAKE_CXX_FLAGS}")

include_directories(
  include
  ${ARMADILLO_INCLUDE_DIRS}
  ${catkin_INCLUDE_DIRS}
)

add_library(libtraj_gen lib/libtraj_gen.cpp)
target_link_libraries(libtraj_gen ${catkin_LIBRARIES} ${ARMADILLO_LIBRARIES})

add_executable(lattice_trajectory_gen nodes/lattice_trajectory_gen/lattice_trajectory_gen.cpp)
target_link_libraries(lattice_trajectory_gen libtraj_gen ${catkin_LIBRARIES})
add_dependencies(lattice_trajectory_gen ${catkin_EXPORTED_TARGETS})

add_executable(lattice_twist_convert nodes/lattice_twist_convert/lattice_twist_convert.cpp)
target_link_libraries(lattice_twist_convert libtraj_gen ${catkin_LIBRARIES})
add_dependencies(lattice_twist_convert ${catkin_EXPORTED_TARGETS})

add_executable(lattice_velocity_set nodes/lattice_velocity_set/lattice_velocity_set.cpp nodes/lattice_velocity_set/libvelocity_set.cpp)
target_link_libraries(lattice_velocity_set ${catkin_LIBRARIES})
add_dependencies(lattice_velocity_set ${catkin_EXPORTED_TARGETS})

add_executable(path_select nodes/path_select/path_select.cpp)
target_link_libraries(path_select ${catkin_LIBRARIES})
add_dependencies(path_select ${catkin_EXPORTED_TARGETS})

install(
  TARGETS
    libtraj_gen
    lattice_trajectory_gen
    lattice_twist_convert
    lattice_velocity_set
    path_select
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

install(
  DIRECTORY launch/
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/launch
)
