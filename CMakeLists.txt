# Copyright (c) 2016 PaddlePaddle Authors. All Rights Reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License

cmake_minimum_required(VERSION 3.0)
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
set(PADDLE_SERVING_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR})
set(PADDLE_SERVING_BINARY_DIR ${CMAKE_CURRENT_BINARY_DIR})

include(system)

project(paddle-serving CXX C)
message(STATUS "CXX compiler: ${CMAKE_CXX_COMPILER}, version: "
        "${CMAKE_CXX_COMPILER_ID} ${CMAKE_CXX_COMPILER_VERSION}")
message(STATUS "C compiler: ${CMAKE_C_COMPILER}, version: "
        "${CMAKE_C_COMPILER_ID} ${CMAKE_C_COMPILER_VERSION}")

find_package(Git REQUIRED)
find_package(Threads REQUIRED)

include(simd)

# CMAKE_BUILD_TYPE
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE "RelWithDebInfo" CACHE STRING
      "Choose the type of build, options are: Debug Release RelWithDebInfo MinSizeRel"
      FORCE)
endif()

set(THIRD_PARTY_PATH "${CMAKE_BINARY_DIR}/third_party" CACHE STRING
  "A path setting third party libraries download & build directories.")

set(FLUID_INSTALL_DIR "${CMAKE_BINARY_DIR}/fluid_install_dir" CACHE STRING
  "A path setting fluid shared and static libraries")

set(FLUID_INFERENCE_INSTALL_DIR "${CMAKE_BINARY_DIR}/fluid_inference_install_dir" CACHE STRING
  "A path setting fluid inference shared and static libraries")

set(THIRD_PARTY_BUILD_TYPE Release)

option(WITH_AVX         "Compile PaddlePaddle with AVX intrinsics"      ${AVX_FOUND})
option(WITH_MKL         "Compile PaddlePaddle with MKL support."        ${AVX_FOUND})

set(WITH_MKLML ${WITH_MKL})
if (NOT DEFINED WITH_MKLDNN)
    if (WITH_MKL AND AVX2_FOUND)
        set(WITH_MKLDNN ON)
    else()
        message(STATUS "Do not have AVX2 intrinsics and disabled MKL-DNN")
        set(WITH_MKLDNN OFF)
    endif()
endif()

include(external/zlib)
include(external/gflags)
include(external/glog)
include(external/leveldb)
include(external/protobuf)
include(external/snappy)
include(external/brpc)
include(external/pcre)
include(external/boost)
include(flags)
include(configure)
include(generic)
include(paddlepaddle)

include_directories("${PADDLE_SERVING_SOURCE_DIR}")

set(EXTERNAL_LIBS
    gflags
    glog
    protobuf
    paddlepaddle
    pcre
    brpc
)

if(WITH_MKLML)
    list(APPEND EXTERNAL_LIBS ${MKLML_IOMP_LIB})
endif()


if(WITH_MKLDNN)
    list(APPEND EXTERNAL_LIBS ${MKLDNN_LIB})
endif()

add_subdirectory(bsl)
add_subdirectory(ullib)
add_subdirectory(spreg)
add_subdirectory(configure)
add_subdirectory(mempool)
add_subdirectory(predictor)