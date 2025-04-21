cmake_minimum_required(VERSION 3.6)
project(hello-physx-cmake LANGUAGES CXX)

# Include the FetchContent module
include(FetchContent)

# DEBUG - if you want to use a local git repository instead of the remote github one, set this to your local checkout path:
set(FETCHCONTENT_SOURCE_DIR_PHYSX_LIB "C:/Users/marco/physx-cmake/PhysX")

# Show download progress in the terminal, if you want
set(FETCHCONTENT_QUIET OFF)

# Declare the external dependency
# The name 'physx_lib' is how we'll refer to it in our project
# Use GIT_REPOSITORY to specify the GitHub repository URL
# Use GIT_TAG to specify a specific commit, tag, or branch
FetchContent_Declare(
    physx_lib
    GIT_REPOSITORY https://github.com/marcodiiga/PhysX.git
    GIT_TAG        main
    SOURCE_SUBDIR  physx
)

# Add BEFORE FetchContent_MakeAvailable
# set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
# set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
# set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
# CMake 4.0+ no longer supports compatibility with < 3.5, so we need to set the policy version
set(CMAKE_POLICY_VERSION_MINIMUM 3.5)

# Make the dependency available to our project
# This automatically downloads the repository, configures it,
# and makes its targets available to our project - it doesn't build PhysX yet!
FetchContent_MakeAvailable(physx_lib)

# Print out the source/binary directories for debugging
message(STATUS "YEAH PhysX source directory: ${physx_lib_SOURCE_DIR}")
message(STATUS "YEAH PhysX binary directory: ${physx_lib_BINARY_DIR}")

# Create an executable for our application
add_executable(hello-physx-cmake main.cpp)

# Link our executable with the physx_lib library
# This now includes all necessary dependencies (PhysX libraries, CUDA, PhysXGpu)!
# Users no longer need to manually specify anything:
# all include directories, compile definitions, and other properties are inherited automatically
target_link_libraries(hello-physx-cmake PRIVATE physx_lib)
