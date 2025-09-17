cmake_minimum_required(VERSION 3.5)
project(joy)

# Load catkin and all dependencies required for this package
set(CATKIN_DEPS roscpp diagnostic_updater sensor_msgs roslint)
find_package(catkin REQUIRED ${CATKIN_DEPS})
roslint_cpp()
catkin_package(DEPENDS sensor_msgs)

include_directories(msg/cpp ${catkin_INCLUDE_DIRS})

if(CATKIN_ENABLE_TESTING)
  roslint_add_test()
endif()

if(${CMAKE_SYSTEM_NAME} STREQUAL "Linux")
  # Look for <linux/joystick.h>
  include(CheckIncludeFiles)
  check_include_files(linux/joystick.h HAVE_LINUX_JOYSTICK_H)
endif()

if(${CMAKE_SYSTEM_NAME} STREQUAL "Linux" AND HAVE_LINUX_JOYSTICK_H)
  add_executable(joy_node src/joy_node.cpp)
  target_link_libraries(joy_node ${catkin_LIBRARIES})
else()
  find_package(SDL2 REQUIRED)
  add_executable(joy_node src/joy_node_sdl.cpp)

  set_target_properties(SDL2::SDL2 PROPERTIES
    MAP_IMPORTED_CONFIG_MINSIZEREL Release
    MAP_IMPORTED_CONFIG_RELWITHDEBINFO Release
    )

  set_target_properties(SDL2::SDL2main PROPERTIES
    MAP_IMPORTED_CONFIG_MINSIZEREL Release
    MAP_IMPORTED_CONFIG_RELWITHDEBINFO Release
    )

  target_link_libraries(joy_node 
    ${catkin_LIBRARIES}
    SDL2::SDL2 SDL2::SDL2main
    )

  file(DOWNLOAD
      https://raw.githubusercontent.com/mdqinc/SDL_GameControllerDB/master/gamecontrollerdb.txt
      ${PROJECT_SOURCE_DIR}/launch/gamecontrollerdb.txt
      SHOW_PROGRESS
  )    

  install(DIRECTORY launch/
    DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/launch
    FILES_MATCHING PATTERN "*.launch"
  )    
endif()

install(TARGETS joy_node
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION})

install(DIRECTORY migration_rules
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
  )