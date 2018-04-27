cmake_minimum_required(VERSION 3.9)

if(UNIX AND NOT APPLE)
    set(LINUX TRUE)
endif()

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_AUTORCC ON)

project(libhuestacean)

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

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_CURRENT_SOURCE_DIR}/cmake/")
message(STATUS "************ ${CMAKE_MODULE_PATH}")

# Proto file
get_filename_component(RPC_PROTO "proto/rpc.proto" ABSOLUTE)
get_filename_component(PROTO_PATH "${RPC_PROTO}" PATH)

file(GLOB ProtoFiles "${CMAKE_CURRENT_SOURCE_DIR}/proto/*.proto")

include(FindGRPC REQUIRED)
include(FindPbuf REQUIRED)
SET(PROTOBUF_IMPORT_DIRS "${CMAKE_CURRENT_SOURCE_DIR}/proto")
SET(PROTOBUF_GENERATE_CPP_APPEND_PATH 1)
PROTOBUF_GEN_CPP(PROTO_SOURCES PROTO_HEADERS ${ProtoFiles})
PROTOBUF_GENERATE_GRPC_CPP(GRPC_SOURCES GRPC_HEADERS ${ProtoFiles})
#############################################

include_directories(
	"${CMAKE_CURRENT_BINARY_DIR}"
	"${CMAKE_CURRENT_SOURCE_DIR}/include"
)

set(SOURCES
   ${SOURCES}
   ${CMAKE_CURRENT_SOURCE_DIR}/src/utility.cpp
)
set(HEADERS
   ${HEADERS}
   ${CMAKE_CURRENT_SOURCE_DIR}/include/utility.h
)

#c/o squareskittles https://stackoverflow.com/questions/9742003/platform-detection-in-cmake
if (WIN32)
    macro(get_WIN32_WINNT version)
        if (CMAKE_SYSTEM_VERSION)
            set(ver ${CMAKE_SYSTEM_VERSION})
            string(REGEX MATCH "^([0-9]+).([0-9])" ver ${ver})
            string(REGEX MATCH "^([0-9]+)" verMajor ${ver})
            # Check for Windows 10, b/c we'll need to convert to hex 'A'.
            if ("${verMajor}" MATCHES "10")
                set(verMajor "A")
                string(REGEX REPLACE "^([0-9]+)" ${verMajor} ver ${ver})
            endif ("${verMajor}" MATCHES "10")
            # Remove all remaining '.' characters.
            string(REPLACE "." "" ver ${ver})
            # Prepend each digit with a zero.
            string(REGEX REPLACE "([0-9A-Z])" "0\\1" ver ${ver})
            set(${version} "0x${ver}")
        endif(CMAKE_SYSTEM_VERSION)
    endmacro(get_WIN32_WINNT)

    get_WIN32_WINNT(ver)
    add_definitions(-D_WIN32_WINNT=${ver})
endif(WIN32)

if(NOT HUESTACEAN_NO_CLIENT)
	add_library(libhuestacean STATIC
		${PROTO_SOURCES}
		${GRPC_SOURCES}
		${PROTO_HEADERS}
		${GRPC_HEADERS}
		${HEADERS}
		${SOURCES}
	)
	target_link_libraries(libhuestacean
		${_GRPC_GRPCPP_UNSECURE}
		${_PROTOBUF_LIBPROTOBUF}
	)
else()
	add_library(libhuestacean STATIC
		${HEADERS}
		${SOURCES}
	)
endif()