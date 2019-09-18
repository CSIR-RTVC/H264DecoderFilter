# CMakeLists.txt for <H264DecoderFilter> 
cmake_minimum_required(VERSION 3.11)

# hints from https://github.com/robotology/how-to-export-cpp-library/blob/master/CMakeLists.txt
project(H264DecoderFilter
        LANGUAGES CXX C
        VERSION 0.0.0.0)

#Make sure that custom modules like FindRapidJSON are found
list(INSERT CMAKE_MODULE_PATH 0 ${CMAKE_SOURCE_DIR}/cmake)

# SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -D_BUILD_FOR_SHORT")
SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -D_CRT_SECURE_NO_WARNINGS")
include(FetchContent)
FetchContent_Declare(
  H264v2
  GIT_REPOSITORY https://github.com/CSIR-RTVC/H264v2
)
# Declare dependencies
find_package(H264v2 1.0.0 REQUIRED)

FetchContent_Declare(
  DirectShowExt
  GIT_REPOSITORY https://github.com/CSIR-RTVC/DirectShowExt
)
# Declare dependencies
find_package(DirectShowExt 1.0.0 REQUIRED)

SET(FLT_HDRS
H264DecoderFilter.h
stdafx.h
VersionInfo.h
)

SET(FLT_SRCS 
DLLSetup.cpp
H264DecoderFilter.cpp
H264DecoderFilter.def
stdafx.cpp
)

ADD_LIBRARY(
H264DecoderFilter SHARED ${FLT_SRCS} ${FLT_HDRS})

target_include_directories(H264DecoderFilter
    PUBLIC 
        $<INSTALL_INTERFACE:install>    
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}
)

LINK_DIRECTORIES(
    $ENV{MSSDK}/Lib
)

TARGET_LINK_LIBRARIES (
H264DecoderFilter
H264v2::H264v2
DirectShowExt::DirectShowExt
) 

INSTALL(
  TARGETS H264DecoderFilter
  ARCHIVE DESTINATION bin
  LIBRARY DESTINATION bin
)

IF (REGISTER_DS_FILTERS)
ADD_CUSTOM_COMMAND(
TARGET H264DecoderFilter 
POST_BUILD COMMAND 
regsvr32 /s \"$(TargetPath)\"
)
ENDIF (REGISTER_DS_FILTERS)
