cmake_minimum_required(VERSION 2.8.3)
project(laser_geometry)

find_package(catkin REQUIRED
    COMPONENTS
        angles
        cmake_modules
        roscpp
        sensor_msgs
        tf
        tf2
)

find_package(Boost REQUIRED COMPONENTS system thread)

find_package(Eigen3 QUIET)
if(NOT EIGEN3_FOUND)
  # Fallback to cmake_modules
  find_package(cmake_modules REQUIRED)
  find_package(Eigen REQUIRED)
  set(EIGEN3_INCLUDE_DIRS ${EIGEN_INCLUDE_DIRS})
  set(EIGEN3_LIBRARIES ${EIGEN_LIBRARIES})  # Not strictly necessary as Eigen is head only
  # Possibly map additional variables to the EIGEN3_ prefix.
else()
  set(EIGEN3_INCLUDE_DIRS ${EIGEN3_INCLUDE_DIR})
endif()

catkin_python_setup()

catkin_package(
  INCLUDE_DIRS include ${EIGEN3_INCLUDE_DIRS}
  LIBRARIES laser_geometry
  DEPENDS Boost Eigen3
)

include_directories(include
    ${catkin_INCLUDE_DIRS}
    ${Boost_INCLUDE_DIRS}
    ${EIGEN3_INCLUDE_DIRS}
)

add_library(laser_geometry src/laser_geometry.cpp)
target_link_libraries(laser_geometry ${Boost_LIBRARIES} ${tf_LIBRARIES})

if(CATKIN_ENABLE_TESTING)
  catkin_add_gtest(projection_test test/projection_test.cpp)
  target_link_libraries(projection_test laser_geometry)

  catkin_add_nosetests(test/projection_test.py)
endif()

install(TARGETS laser_geometry
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_GLOBAL_BIN_DESTINATION})
install(DIRECTORY include/laser_geometry/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
  FILES_MATCHING PATTERN "*.h")
