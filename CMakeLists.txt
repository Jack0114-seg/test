cmake_minimum_required(VERSION 3.1.0)
project(Pysot
        VERSION 1.0.0)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -Wl,--allow-shlib-undefined")

set(CMAKE_INSTALL_PREFIX ${CMAKE_SOURCE_DIR}/install/${CMAKE_SYSTEM_NAME})

set(CMAKE_SKIP_INSTALL_RPATH FALSE)
set(CMAKE_BUILD_WITH_INSTALL_RPATH TRUE)
set(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib")

set(LIB_ARCH aarch64)

include_directories(${CMAKE_SOURCE_DIR})

#rknn api
set(RKNN_API_PATH ${CMAKE_SOURCE_DIR}/lib/rknn)
if(CMAKE_SYSTEM_NAME STREQUAL "Android")
    set(RKNN_RT_LIB ${RKNN_API_PATH}/${CMAKE_ANDROID_ARCH_ABI}/librknnrt.so)
else()
    set(RKNN_RT_LIB ${RKNN_API_PATH}/librknnrt.so)
endif()

include_directories(${RKNN_API_PATH}/include/rknn)
include_directories(${CMAKE_SOURCE_DIR}/third)

#opencv
if(CMAKE_SYSTEM_NAME STREQUAL "Android")
  set(OpenCV_DIR ${CMAKE_SOURCE_DIR}/../3rdparty/opencv/OpenCV-android-sdk/sdk/native/jni/abi-${CMAKE_ANDROID_ARCH_ABI})
else()
  if(LIB_ARCH STREQUAL "armhf")
    set(OpenCV_DIR ${CMAKE_SOURCE_DIR}/../3rdparty/opencv/opencv-linux-armhf/share/OpenCV)
  else()
    set(OpenCV_DIR ${CMAKE_SOURCE_DIR}/third/opencv/opencv-linux-aarch64/share/OpenCV)
  endif()
endif()

find_package(OpenCV REQUIRED)
#rga
# comes from https://github.com/airockchip/librga
set(RGA_PATH ${CMAKE_SOURCE_DIR}/third/rga)
if(CMAKE_SYSTEM_NAME STREQUAL "Android")
  set(RGA_LIB ${RGA_PATH}/libs/AndroidNdk/${CMAKE_ANDROID_ARCH_ABI}/librga.so)
else()
  message("---build environment---" ${CMAKE_C_COMPILER})

  set(LIB_ARCH aarch64)

  set(RGA_LIB ${RGA_PATH}/libs/Linux/gcc-${LIB_ARCH}/librga.so)
endif()
include_directories( ${RGA_PATH}/include)

# mpp
set(MPP_PATH ${CMAKE_CURRENT_SOURCE_DIR}/third/mpp)

if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
  set(MPP_LIBS ${MPP_PATH}/${CMAKE_SYSTEM_NAME}/${LIB_ARCH}/librockchip_mpp.so)
elseif(CMAKE_SYSTEM_NAME STREQUAL "Android")
  set(MPP_LIBS ${MPP_PATH}/${CMAKE_SYSTEM_NAME}/${CMAKE_ANDROID_ARCH_ABI}/libmpp.so)
endif()

include_directories(${MPP_PATH}/include)

# zlmediakit
set(ZLMEDIAKIT_PATH ${CMAKE_SOURCE_DIR}/third/zlmediakit)

if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
  include_directories(${ZLMEDIAKIT_PATH}/include)
  set(ZLMEDIAKIT_LIBS ${ZLMEDIAKIT_PATH}/${LIB_ARCH}/libmk_api.so)
endif()

if(ZLMEDIAKIT_LIBS)
  add_definitions(-DBUILD_VIDEO_RTSP)
endif()

set(CMAKE_INSTALL_RPATH "lib")

include_directories(${CMAKE_SOURCE_DIR}/include)

add_executable(test_rknn_lib
  src/main.cc
  src/preprocess.cc
  src/postprocess.cc
)

target_link_libraries(test_rknn_lib
  ${RKNN_RT_LIB}
  ${RGA_LIB}
  ${OpenCV_LIBS}
)

# if(MPP_LIBS)
#   add_executable(rknn_yolov5_video_demo
#     src/main_video.cc
#     src/postprocess.cc
#     utils/mpp_decoder.cpp
#     utils/mpp_encoder.cpp
#     utils/drawing.cpp
#   )
#   target_link_libraries(rknn_yolov5_video_demo
#     ${RKNN_RT_LIB}
#     ${RGA_LIB}
#     ${OpenCV_LIBS}
#     ${MPP_LIBS}
#     ${ZLMEDIAKIT_LIBS}
#   )
# endif()
