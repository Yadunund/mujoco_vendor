
cmake_minimum_required(VERSION 3.8)
project(mujoco_vendor)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

find_package(ament_cmake REQUIRED)
find_package(ament_cmake_vendor_package REQUIRED)
include(ExternalProject)

set(MUJOCO_BUILD_BRANCH 3.5.0)
ament_vendor(mujoco_vendor
  VCS_URL https://github.com/google-deepmind/mujoco.git
  VCS_VERSION ${MUJOCO_BUILD_BRANCH}
  CMAKE_ARGS
    "-DCMAKE_BUILD_TYPE=Release"
    "-DCMAKE_CXX_FLAGS=-flto=auto"
    "-DCMAKE_C_FLAGS=-flto=auto"
    GLOBAL_HOOK
)
ExternalProject_Get_Property(mujoco_vendor SOURCE_DIR)
ExternalProject_Get_Property(mujoco_vendor BINARY_DIR)

install(DIRECTORY ${SOURCE_DIR}/simulate/
  DESTINATION opt/${PROJECT_NAME}/simulate
)

# Install MuJoCo plugin libraries to lib/ (for ros2_control and other C++ code)
install(
  FILES
    ${BINARY_DIR}/lib/libelasticity.so
    ${BINARY_DIR}/lib/libactuator.so
    ${BINARY_DIR}/lib/libsensor.so
    ${BINARY_DIR}/lib/libsdf_plugin.so
  DESTINATION opt/${PROJECT_NAME}/lib
  OPTIONAL
)

# Install plugin libraries to bin/mujoco_plugin/ (for simulate binary)
install(
  FILES
    ${BINARY_DIR}/lib/libelasticity.so
    ${BINARY_DIR}/lib/libactuator.so
    ${BINARY_DIR}/lib/libsensor.so
    ${BINARY_DIR}/lib/libsdf_plugin.so
  DESTINATION opt/${PROJECT_NAME}/bin/mujoco_plugin
  OPTIONAL
)

# Install plugin model files
install(DIRECTORY ${SOURCE_DIR}/model/plugin/
  DESTINATION opt/${PROJECT_NAME}/share/mujoco/model/plugin
  OPTIONAL
)

# Install environment hook to set MUJOCO_DIR
ament_environment_hooks(hooks/mujoco_vendor.sh.in)

ament_package(
  CONFIG_EXTRAS_POST "mujoco_vendor-extra.cmake.in"
)
