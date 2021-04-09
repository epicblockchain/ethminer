# A custom command and target to turn the OpenCL kernel into a byte array header
# The normal build depends on it properly and if the kernel file is changed, then
# a rebuild of libethash-cl should be triggered

#TODO: clean up the copy&pasting here
add_custom_command(
	OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/sha3.h
	COMMAND ${CMAKE_COMMAND} ARGS
	-DBIN2H_SOURCE_FILE="${CMAKE_CURRENT_SOURCE_DIR}/kernels/cl/sha3.cl"
	-DBIN2H_VARIABLE_NAME=sha3_cl
	-DBIN2H_HEADER_FILE="${CMAKE_CURRENT_BINARY_DIR}/sha3.h"
	-P "${CMAKE_CURRENT_SOURCE_DIR}/bin2h.cmake"
	COMMENT "Generating OpenCL Kernel Byte Array"
	DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/kernels/cl/sha3.cl
)
add_custom_target(sha3clbin2h_stable DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/sha3.h ${CMAKE_CURRENT_SOURCE_DIR}/cl/sha3.cl)

set(SOURCES
	Sha3CLMiner.h Sha3CLMiner.cpp
	${CMAKE_CURRENT_BINARY_DIR}/sha3.h
)

if(USE_SYS_OPENCL)
	# On macOS or using ROCm-OpenCL, use system OpenCL library.
	find_package(OpenCL REQUIRED)
else()
	hunter_add_package(OpenCL)
	find_package(OpenCL CONFIG REQUIRED)
endif()

include_directories(${CMAKE_CURRENT_BINARY_DIR})
include_directories(..)

add_library(sha3-cl ${SOURCES})
target_link_libraries(sha3-cl PUBLIC ethcore)
target_link_libraries(sha3-cl PRIVATE OpenCL::OpenCL)
target_link_libraries(sha3-cl PRIVATE Boost::filesystem Boost::thread)
