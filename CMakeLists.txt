CMAKE_MINIMUM_REQUIRED(VERSION 3.7 FATAL_ERROR)
project(cbf)

list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)
set(CMAKE_CXX_STANDARD 20)
add_definitions(-DANTLR4CPP_STATIC)

set(ANTLR4_WITH_STATIC_CRT OFF)
set(ANTLR4_TAG 4.9.3)

include(ExternalAntlr4Cpp)
set(ANTLR_EXECUTABLE /usr/local/lib/antlr-4.9.3-complete.jar)
set(ANTLR4_RUNTIME_LIBRARIES /usr/local/lib)
include_directories(${ANTLR4_INCLUDE_DIRS})

find_package(ANTLR REQUIRED)
find_package(LLVM 14 REQUIRED CONFIG)

message(STATUS "Found LLVM ${LLVM_PACKAGE_VERSION}")
message(STATUS "Using LLVMConfig.cmake in: ${LLVM_DIR}")

# Set your project compile flags.
# E.g. if using the C++ header files
# you will need to enable C++11 support
# for your compiler.

include_directories(${LLVM_INCLUDE_DIRS})
add_definitions(${LLVM_DEFINITIONS})
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")

antlr_target(CTarget brainfuck.g4 VISITOR)
include_directories(${ANTLR_CTarget_OUTPUT_DIR})

set(SOURCES
	src/main.cpp

	src/codegenState.cpp
	src/brainfuckState.cpp

	src/ast/advanceCellNode.cpp
)
set(HEADERS
	include/codegenState.hpp
	include/brainfuckState.hpp

	include/ast/astNode.hpp
	include/ast/advanceCellNode.hpp
)

# Now build our tools
add_executable(${PROJECT_NAME}
	${HEADERS}
	${SOURCES}
	${ANTLR_CTarget_CXX_OUTPUTS}
)
target_include_directories(${PROJECT_NAME} PUBLIC . include/)

# Find the libraries that correspond to the LLVM components
# that we wish to use
llvm_map_components_to_libnames(llvm_libs Support Core IRReader)

message(STATUS "Using LLVM libs: ${llvm_libs}")

# Link against LLVM libraries
target_link_libraries(cbf antlr4_static ${llvm_libs})
