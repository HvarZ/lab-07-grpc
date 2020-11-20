cmake_minimum_required(VERSION 3.4)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

option(BUILD_COVERAGE "Build coverage" OFF)

set(
  HUNTER_CACHE_SERVERS
  "https://github.com/bmstu-iu8-cpp-sem-3/hunter-cache"
  CACHE STRING "Default cache server"
)

set(HUNTER_CONFIGURATION_TYPES Release)
set(HUNTER_BUILD_SHARED_LIBS ON)

include("tools/gate/cmake/HunterGate.cmake")

HunterGate(
  URL "https://github.com/cpp-pm/hunter/archive/v0.23.278.tar.gz"
  SHA1 "35fa55bc1dfcf9f4d7264f8bb26cd1759ce1ae07"
)

project(echoservice)
string(APPEND CMAKE_CXX_FLAGS " -pedantic -Werror -Wall -Wextra")
string(APPEND CMAKE_CXX_FLAGS " -Wno-unused-command-line-argument -Wno-unused-parameter")
string(APPEND CMAKE_CXX_FLAGS " -Wshadow -Wnon-virtual-dtor")

hunter_add_package(GTest)
find_package(GTest CONFIG REQUIRED)

hunter_add_package(gRPC)
find_package(gRPC CONFIG REQUIRED)

hunter_add_package(Protobuf)
find_package(Protobuf CONFIG REQUIRED)

set(PROTO_GENERATED_SOURCES
  "${CMAKE_CURRENT_BINARY_DIR}/echo.pb.cc"
  "${CMAKE_CURRENT_BINARY_DIR}/google/api/annotations.pb.cc"
  "${CMAKE_CURRENT_BINARY_DIR}/google/api/http.pb.cc"
  "${CMAKE_CURRENT_BINARY_DIR}/google/rpc/status.pb.cc")

set(PROTO_GENERATED_HEADERS
  "${CMAKE_CURRENT_BINARY_DIR}/echo.pb.h"
  "${CMAKE_CURRENT_BINARY_DIR}/google/api/annotations.pb.h"
  "${CMAKE_CURRENT_BINARY_DIR}/google/api/http.pb.h"
  "${CMAKE_CURRENT_BINARY_DIR}/google/rpc/status.pb.h")

set(PROTO_GENERATED_GRPC_SOURCES
  "${CMAKE_CURRENT_BINARY_DIR}/echo.grpc.pb.cc"
  "${CMAKE_CURRENT_BINARY_DIR}/google/api/annotations.grpc.pb.cc"
  "${CMAKE_CURRENT_BINARY_DIR}/google/api/http.grpc.pb.cc"
  "${CMAKE_CURRENT_BINARY_DIR}/google/rpc/status.grpc.pb.cc")

set(PROTO_GENERATED_GRPC_HEADERS
  "${CMAKE_CURRENT_BINARY_DIR}/echo.grpc.pb.h"
  "${CMAKE_CURRENT_BINARY_DIR}/google/api/annotations.grpc.pb.h"
  "${CMAKE_CURRENT_BINARY_DIR}/google/api/http.grpc.pb.h"
  "${CMAKE_CURRENT_BINARY_DIR}/google/rpc/status.grpc.pb.h")

find_program(PROTOBUF_PROTOC protoc)
find_program(GRPC_CPP_PLUGIN_EXECUTABLE grpc_cpp_plugin)

set(PROTO_FILES
  "${CMAKE_CURRENT_SOURCE_DIR}/protos/echo.proto"
  "${CMAKE_CURRENT_SOURCE_DIR}/protos/google/api/annotations.proto"
  "${CMAKE_CURRENT_SOURCE_DIR}/protos/google/api/http.proto"
  "${CMAKE_CURRENT_SOURCE_DIR}/protos/google/rpc/status.proto")

set(PROTO_INCLUDE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/protos")
add_custom_command(
      OUTPUT  ${PROTO_GENERATED_SOURCES}
              ${PROTO_GENERATED_HEADERS}
              ${PROTO_GENERATED_GRPC_SOURCES}
              ${PROTO_GENERATED_GRPC_HEADERS}
      COMMAND protobuf::protoc
      ARGS
        --grpc_out "${CMAKE_CURRENT_BINARY_DIR}"
        --cpp_out "${CMAKE_CURRENT_BINARY_DIR}"
        -I"${PROTO_INCLUDE_DIR}"
        --plugin=protoc-gen-grpc="${GRPC_CPP_PLUGIN_EXECUTABLE}"
        ${PROTO_FILES}
      DEPENDS ${PROTO_FILES})

add_executable(grpc_service
  ${CMAKE_CURRENT_SOURCE_DIR}/sources/main.cpp
  ${PROTO_GENERATED_SOURCES}
  ${PROTO_GENERATED_GRPC_SOURCES})

target_include_directories(grpc_service PRIVATE "${CMAKE_CURRENT_BINARY_DIR}")

target_link_libraries(grpc_service
  gRPC::grpc++ gRPC::grpc++_reflection
  protobuf::libprotobuf)
