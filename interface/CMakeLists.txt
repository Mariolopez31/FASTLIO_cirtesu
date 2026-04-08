cmake_minimum_required(VERSION 3.8)
project(interface)

# Detect Raspberry-like architectures and skip build there
set(TARGET_RASPBERRY FALSE)
if(CMAKE_SYSTEM_PROCESSOR MATCHES "armv7|aarch64")
  set(TARGET_RASPBERRY TRUE)
endif()

if(TARGET_RASPBERRY)
  message(WARNING
    "Package '${PROJECT_NAME}' will NOT be compiled because a Raspberry/ARM "
    "architecture was detected "
    "(CMAKE_SYSTEM_PROCESSOR='${CMAKE_SYSTEM_PROCESSOR}')."
  )
  find_package(ament_cmake REQUIRED)
  ament_package()
  return()
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# find dependencies
find_package(ament_cmake REQUIRED)
find_package(std_msgs REQUIRED)
find_package(rosidl_default_generators REQUIRED)

rosidl_generate_interfaces(${PROJECT_NAME}
  "srv/SaveMaps.srv"
  "srv/Relocalize.srv"
  "srv/IsValid.srv"
  "srv/RefineMap.srv"
  "srv/SavePoses.srv"
  DEPENDENCIES std_msgs
)



if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  # the following line skips the linter which checks for copyrights
  # comment the line when a copyright and license is added to all source files
  set(ament_cmake_copyright_FOUND TRUE)
  # the following line skips cpplint (only works in a git repo)
  # comment the line when this package is in a git repo and when
  # a copyright and license is added to all source files
  set(ament_cmake_cpplint_FOUND TRUE)
  ament_lint_auto_find_test_dependencies()
endif()

ament_package()
