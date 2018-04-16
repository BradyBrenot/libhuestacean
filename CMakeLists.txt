cmake_minimum_required(VERSION 3.9)

if(UNIX AND NOT APPLE)
    set(LINUX TRUE)
endif()

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_AUTORCC ON)

#############################################
# Gen protocol buffer and grpc code.
# Following gRPC helloworld CMake example

if(NO_GRPC_SUBMODULE)
	# Find Protobuf installation
	# Looks for protobuf-config.cmake file installed by Protobuf's cmake installation.
	set(protobuf_MODULE_COMPATIBLE TRUE)
	find_package(Protobuf CONFIG REQUIRED)
	message(STATUS "Using protobuf ${protobuf_VERSION}")

	set(_PROTOBUF_LIBPROTOBUF protobuf::libprotobuf)
	set(_PROTOBUF_PROTOC $<TARGET_FILE:protobuf::protoc>)

	# Find gRPC installation
	# Looks for gRPCConfig.cmake file installed by gRPC's cmake installation.
	find_package(gRPC CONFIG REQUIRED)
	message(STATUS "Using gRPC ${gRPC_VERSION}")

	set(_GRPC_GRPCPP_UNSECURE gRPC::grpc++_unsecure)
	set(_GRPC_CPP_PLUGIN_EXECUTABLE $<TARGET_FILE:gRPC::grpc_cpp_plugin>)
else()
	add_subdirectory(grpc EXCLUDE_FROM_ALL)
	message(STATUS "Using gRPC via add_subdirectory.")

	set(_PROTOBUF_LIBPROTOBUF libprotobuf)
	set(PROTOBUF_PROTOC_EXECUTABLE $<TARGET_FILE:protoc>)
	set(_GRPC_GRPCPP_UNSECURE grpc++_unsecure)
	set(GRPC_CPP_PLUGIN $<TARGET_FILE:grpc_cpp_plugin>)
endif()

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/")

# Proto file
get_filename_component(RPC_PROTO "proto/rpc.proto" ABSOLUTE)
get_filename_component(PROTO_PATH "${RPC_PROTO}" PATH)

file(GLOB ProtoFiles "${CMAKE_CURRENT_SOURCE_DIR}/proto/*.proto")

include(FindPbuf REQUIRED)
SET(PROTOBUF_IMPORT_DIRS "${CMAKE_CURRENT_SOURCE_DIR}/proto")
SET(PROTOBUF_GENERATE_CPP_APPEND_PATH 1)
PROTOBUF_GEN_CPP(PROTO_SOURCES PROTO_HEADERS ${ProtoFiles})
PROTOBUF_GENERATE_GRPC_CPP(GRPC_SOURCES GRPC_HEADERS ${ProtoFiles})

include_directories("${CMAKE_CURRENT_BINARY_DIR}")
#############################################

set(SOURCES
   ${SOURCES}
   ${CMAKE_CURRENT_SOURCE_DIR}/src/utility.cpp
)
set(HEADERS
   ${HEADERS}
   ${CMAKE_CURRENT_SOURCE_DIR}/include/utility.h
)

add_library(libhuestacean STATIC
	${PROTO_SOURCES}
	${GRPC_SOURCES}
	${PROTO_HEADERS}
	${GRPC_HEADERS}
	${HEADERS}
)
target_link_libraries(libhuestacean
	${_GRPC_GRPCPP_UNSECURE}
	${_PROTOBUF_LIBPROTOBUF}
)

project(libhuestacean)