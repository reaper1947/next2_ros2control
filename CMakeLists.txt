cmake_minimum_required(VERSION 3.16)
project(next2_ros2control)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# set the same behavior for windows as it is on linux
set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)

# find dependencies
set(THIS_PACKAGE_INCLUDE_DEPENDS
  hardware_interface
  pluginlib
  rclcpp
  rclcpp_lifecycle
)

# find dependencies
find_package(ament_cmake REQUIRED)
find_package(rclpy REQUIRED)
find_package(rclcpp REQUIRED)
find_package(backward_ros REQUIRED)
foreach(Dependency IN ITEMS ${THIS_PACKAGE_INCLUDE_DEPENDS})
  find_package(${Dependency} REQUIRED)
endforeach()


add_library(
  next2_ros2control
  SHARED
  hardware/diffbot_system.cpp
)
target_compile_features(next2_ros2control PUBLIC cxx_std_17)
target_include_directories(next2_ros2control PUBLIC
$<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/hardware/include>
$<INSTALL_INTERFACE:include/next2_ros2control>
)

ament_target_dependencies(
  next2_ros2control PUBLIC
  ${THIS_PACKAGE_INCLUDE_DEPENDS}
)

# Export hardware plugins
pluginlib_export_plugin_description_file(hardware_interface next2_ros2control.xml)

# INSTALL
install(
  DIRECTORY hardware/include/
  DESTINATION include/next2_ros2control
)
install(
  DIRECTORY description/ros2_control description description/urdf
  DESTINATION share/next2_ros2control
)
install(
  DIRECTORY bringup/launch bringup/config
  DESTINATION share/next2_ros2control
)
install(TARGETS next2_ros2control
  EXPORT export_next2_ros2control
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
)


if(BUILD_TESTING)
  find_package(ament_cmake_pytest REQUIRED)

  ament_add_pytest_test(example_2_urdf_xacro test/test_urdf_xacro.py)
  ament_add_pytest_test(view_example_2_launch test/test_view_robot_launch.py)
  ament_add_pytest_test(run_example_2_launch test/test_diffbot_launch.py)
endif()

## EXPORTS
ament_export_targets(export_next2_ros2control HAS_LIBRARY_TARGET)
ament_export_dependencies(${THIS_PACKAGE_INCLUDE_DEPENDS})
ament_package()