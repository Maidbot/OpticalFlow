############################################################################
#
#   Copyright (c) 2017 PX4 Development Team. All rights reserved.
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions
# are met:
#
# 1. Redistributions of source code must retain the above copyright
#    notice, this list of conditions and the following disclaimer.
# 2. Redistributions in binary form must reproduce the above copyright
#    notice, this list of conditions and the following disclaimer in
#    the documentation and/or other materials provided with the
#    distribution.
# 3. Neither the name PX4 nor the names of its contributors may be
#    used to endorse or promote products derived from this software
#    without specific prior written permission.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
# "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
# LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS
# FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE
# COPYRIGHT OWNER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT,
# INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING,
# BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS
# OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED
# AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT
# LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN
# ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
# POSSIBILITY OF SUCH DAMAGE.
#
############################################################################

cmake_minimum_required( VERSION 2.8.3 )
if( POLICY CMP0048 )
  # Silence CMP0048 warning about missing project VERSION.
  cmake_policy(SET CMP0048 NEW)
endif()
if( POLICY CMP0042 )
  cmake_policy(SET CMP0042 NEW)
endif()
project( optical_flow )

find_package(catkin REQUIRED COMPONENTS
  klt_feature_tracker)

catkin_package(
  INCLUDE_DIRS include
  CATKIN_DEPENDS klt_feature_tracker
  LIBRARIES ${PROJECT_NAME}
)

include_directories(
  include
  ${catkin_INCLUDE_DIRS}
)

set(version_major 0)
set(version_minor 1)
set(version_patch 0)
set(version "${version_major}.${version_minor}.${version_patch}")
set(package-contact "christoph@px4.io")

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fpic")

set(${PROJECT_NAME}_INCLUDE_DIRS
    ${PROJECT_SOURCE_DIR}/include
    CACHE INTERNAL "${PROJECT_NAME}: Include Directories" FORCE)

find_package( OpenCV REQUIRED )
OPTION(OPENCV "ON for OpenCV" ON) # -DOPENCV=OFF

if(OpenCV_FOUND AND OPENCV)
    message(STATUS "Building ${PROJECT_NAME} with OpenCV")

    set(${PROJECT_NAME}_INCLUDE_DIRS
      ${catkin_INCLUDE_DIRS}
      ${OpenCV_INCLUDE_DIRS}
    )

    include_directories(${PROJECT_NAME}_INCLUDE_DIRS)

    add_library( ${PROJECT_NAME} SHARED
      src/optical_flow.cpp
      src/px4flow.cpp
     	src/flow_px4.cpp
    	src/flow_opencv.cpp
    )

    target_link_libraries( ${PROJECT_NAME}
      ${OpenCV_LIBS}
      ${catkin_LIBRARIES}
    )
else()
    message(STATUS "Building ${PROJECT_NAME} without OpenCV")

    set(${PROJECT_NAME}_INCLUDE_DIRS
      ${CMAKE_CURRENT_SOURCE_DIR}/include
    )

    include_directories(${PROJECT_NAME}_INCLUDE_DIRS)

    add_library( ${PROJECT_NAME} SHARED
      src/optical_flow.cpp
      src/px4flow.cpp
     	src/flow_px4.cpp
    )

endif()

target_include_directories(${PROJECT_NAME} PUBLIC include)

# Install Targets
install(TARGETS ${PROJECT_NAME}
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_GLOBAL_BIN_DESTINATION}
)

install(DIRECTORY include/${PROJECT_NAME}/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
)