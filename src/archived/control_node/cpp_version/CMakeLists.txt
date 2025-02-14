cmake_minimum_required(VERSION 3.8)
project(control_node)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# Find dependencies
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(std_msgs REQUIRED)

# Ensure lgpio is available
include_directories(${CMAKE_SOURCE_DIR}/usr/include)  # Adjust path if necessary

# Add executable and link dependencies
add_executable(control_node src/control_node.cpp)
include_directories(include)
include_directories(${CMAKE_SOURCE_DIR}/usr/include)

ament_target_dependencies(control_node rclcpp std_msgs)

# Link liblgpio explicitly
find_package(lgpio REQUIRED)
target_link_libraries(control_node PRIVATE lgpio)

# Install the executable
install(TARGETS control_node DESTINATION lib/${PROJECT_NAME})

# Add testing if needed
if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  set(ament_cmake_copyright_FOUND TRUE)
  set(ament_cmake_cpplint_FOUND TRUE)
  ament_lint_auto_find_test_dependencies()
endif()

ament_package()
