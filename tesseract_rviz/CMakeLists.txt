cmake_minimum_required(VERSION 2.8.3)
project(tesseract_rviz)

add_compile_options(-std=c++11 -Wall -Wextra)

find_package(catkin REQUIRED COMPONENTS
  roscpp
  message_generation
  urdf
  srdfdom
  tesseract_ros
  tesseract_msgs
  rviz
  pluginlib
  interactive_markers
  geometric_shapes
)

## System dependencies are found with CMake's conventions
find_package(Boost REQUIRED thread date_time system filesystem)

# Ogre
find_package(PkgConfig REQUIRED)
pkg_check_modules(OGRE OGRE)
link_directories(${OGRE_LIBRARY_DIRS} )

find_package(Eigen3 REQUIRED)

# Eigen 3.2 (Wily) only provides EIGEN3_INCLUDE_DIR, not EIGEN3_INCLUDE_DIRS
if(NOT EIGEN3_INCLUDE_DIRS)
  set(EIGEN3_INCLUDE_DIRS ${EIGEN3_INCLUDE_DIR})
endif()

# Qt Stuff
if(rviz_QT_VERSION VERSION_LESS "5")
  find_package(Qt4 ${rviz_QT_VERSION} REQUIRED QtCore QtGui)
  include(${QT_USE_FILE})
  macro(qt_wrap_ui)
    qt4_wrap_ui(${ARGN})
  endmacro()
  macro(qt_wrap_cpp)
    qt4_wrap_cpp(${ARGN})
  endmacro()
else()
  find_package(Qt5 ${rviz_QT_VERSION} REQUIRED Core Widgets)
  set(QT_LIBRARIES Qt5::Widgets)
  macro(qt_wrap_ui)
    qt5_wrap_ui(${ARGN})
  endmacro()
  macro(qt_wrap_cpp)
    qt5_wrap_cpp(${ARGN})
  endmacro()
endif()

add_definitions(-DQT_NO_KEYWORDS)

catkin_package(
  INCLUDE_DIRS
    include
    ${Boost_INCLUDE_DIRS}
    ${EIGEN3_INCLUDE_DIRS}
    ${QT_INCLUDE_DIR}
    ${OGRE_INCLUDE_DIRS}
  LIBRARIES
    ${PROJECT_NAME}_render_tools
    ${PROJECT_NAME}_state_plugin_core
  CATKIN_DEPENDS
    roscpp
    message_generation
    urdf
    srdfdom
    tesseract_ros
    tesseract_msgs
    rviz
    pluginlib
    interactive_markers
    geometric_shapes
  DEPENDS
    Boost
    OGRE
    QT
)

include_directories(
  include
  ${catkin_INCLUDE_DIRS}
  SYSTEM ${Boost_INCLUDE_DIRS}
  SYSTEM ${EIGEN3_INCLUDE_DIRS}
  SYSTEM ${QT_INCLUDE_DIR}
  SYSTEM ${OGRE_INCLUDE_DIRS}
)

# Rviz Render Tools
qt_wrap_cpp(${PROJECT_NAME}_render_tools_cpp_MOCS
  include/tesseract_rviz/render_tools/trajectory_visualization.h
  include/tesseract_rviz/render_tools/trajectory_panel.h
  include/tesseract_rviz/render_tools/env/robot.h
  include/tesseract_rviz/render_tools/env/robot_link.h
  include/tesseract_rviz/render_tools/env/robot_joint.h
  )

add_library(${PROJECT_NAME}_render_tools
  src/render_tools/env/robot.cpp
  src/render_tools/env/robot_joint.cpp
  src/render_tools/env/robot_link.cpp
  src/render_tools/state_visualization.cpp
  src/render_tools/link_updater.cpp
  src/render_tools/trajectory_visualization.cpp
  src/render_tools/trajectory_panel.cpp
  ${${PROJECT_NAME}_render_tools_cpp_MOCS}
)

target_link_libraries(${PROJECT_NAME}_render_tools
  ${catkin_LIBRARIES}
  ${OGRE_LIBRARIES}
  ${QT_LIBRARIES}
  ${Boost_LIBRARIES}
)

target_compile_options(${PROJECT_NAME}_render_tools PRIVATE -Wsuggest-override -Wconversion -Wsign-conversion)

#add_library(${PROJECT_NAME}_scene_plugin_core
#  src/tesseract_scene_plugin/tesseract_scene_display.cpp
#  include/tesseract_rviz/tesseract_scene_plugin/tesseract_scene_display.h)
#target_link_libraries(${PROJECT_NAME}_scene_plugin_core ${PROJECT_NAME}_render_tools ${catkin_LIBRARIES} ${OGRE_LIBRARIES} ${QT_LIBRARIES} ${Boost_LIBRARIES})

#add_library(${PROJECT_NAME}_scene_plugin src/tesseract_scene_plugin/plugin_init.cpp)
#target_link_libraries(${PROJECT_NAME}_scene_plugin ${PROJECT_NAME}_scene_plugin_core ${catkin_LIBRARIES} ${Boost_LIBRARIES})

# Trajectory State Disply
qt_wrap_cpp(${PROJECT_NAME}_state_plugin_core_cpp_MOCS include/tesseract_rviz/tesseract_state_plugin/tesseract_state_display.h)

add_library(${PROJECT_NAME}_state_plugin_core src/tesseract_state_plugin/tesseract_state_display.cpp ${${PROJECT_NAME}_state_plugin_core_cpp_MOCS})
target_link_libraries(${PROJECT_NAME}_state_plugin_core ${PROJECT_NAME}_render_tools ${catkin_LIBRARIES} ${OGRE_LIBRARIES} ${QT_LIBRARIES} ${Boost_LIBRARIES})
target_compile_options(${PROJECT_NAME}_state_plugin_core PRIVATE -Wsuggest-override -Wconversion -Wsign-conversion)

add_library(${PROJECT_NAME}_state_plugin src/tesseract_state_plugin/plugin_init.cpp)
target_link_libraries(${PROJECT_NAME}_state_plugin ${PROJECT_NAME}_state_plugin_core ${catkin_LIBRARIES} ${Boost_LIBRARIES})
target_compile_options(${PROJECT_NAME}_state_plugin PRIVATE -Wsuggest-override -Wconversion -Wsign-conversion)

# Trajectory Display
qt_wrap_cpp(${PROJECT_NAME}_trajectory_plugin_core_cpp_MOCS include/tesseract_rviz/tesseract_trajectory_plugin/tesseract_trajectory_display.h)

add_library(${PROJECT_NAME}_trajectory_plugin_core src/tesseract_trajectory_plugin/tesseract_trajectory_display.cpp ${${PROJECT_NAME}_trajectory_plugin_core_cpp_MOCS})
target_link_libraries(${PROJECT_NAME}_trajectory_plugin_core ${PROJECT_NAME}_render_tools ${PROJECT_NAME}_state_plugin_core ${catkin_LIBRARIES} ${OGRE_LIBRARIES} ${QT_LIBRARIES} ${Boost_LIBRARIES})
target_compile_options(${PROJECT_NAME}_trajectory_plugin_core PRIVATE -Wsuggest-override -Wconversion -Wsign-conversion)

add_library(${PROJECT_NAME}_trajectory_plugin src/tesseract_trajectory_plugin/plugin_init.cpp)
target_link_libraries(${PROJECT_NAME}_trajectory_plugin ${PROJECT_NAME}_trajectory_plugin_core ${catkin_LIBRARIES} ${Boost_LIBRARIES})
target_compile_options(${PROJECT_NAME}_trajectory_plugin PRIVATE -Wsuggest-override -Wconversion -Wsign-conversion)

add_executable(${PROJECT_NAME}_test src/test.cpp)
target_link_libraries(${PROJECT_NAME}_test ${PROJECT_NAME}_trajectory_plugin_core ${catkin_LIBRARIES})
target_compile_options(${PROJECT_NAME}_test PRIVATE -Wsuggest-override -Wconversion -Wsign-conversion)


# Mark executables and/or libraries for installation
install(TARGETS ${PROJECT_NAME}_render_tools ${PROJECT_NAME}_state_plugin_core ${PROJECT_NAME}_state_plugin ${PROJECT_NAME}_trajectory_plugin_core ${PROJECT_NAME}_trajectory_plugin
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

# Mark cpp header files for installation
install(DIRECTORY include/${PROJECT_NAME}/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
  FILES_MATCHING PATTERN "*.h"
  PATTERN ".svn" EXCLUDE
)

# Mark plugin XML files for installation
install(FILES tesseract_rviz_state_plugin_description.xml tesseract_rviz_trajectory_plugin_description.xml
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)
