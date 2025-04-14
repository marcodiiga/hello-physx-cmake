cmake_minimum_required(VERSION 3.16)
project(hello-physx-cmake LANGUAGES CXX)

# Set C++ standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Include the FetchContent module
include(FetchContent)

# DEBUG - Instead of using GIT_REPOSITORY, set this to your local checkout:
set(FETCHCONTENT_SOURCE_DIR_PHYSX_LIB "/home/alex/fetchcontent_physx/PhysX/physx")

# Declare the external dependency
# The name 'physx_lib' is how we'll refer to it in our project
# Use GIT_REPOSITORY to specify the GitHub repository URL
# Use GIT_TAG to specify a specific commit, tag, or branch
FetchContent_Declare(
    physx_lib
    GIT_REPOSITORY https://github.com/marcodiiga/PhysX.git
#    GIT_TAG        v1.0.0  # Use a specific tag for reproducible builds
)

# Make the dependency available to our project
# This automatically downloads the repository, configures it,
# and makes its targets available to our project
FetchContent_MakeAvailable(physx_lib)

# Print out the source/binary directories for debugging
message(STATUS "PhysX source directory: ${physx_lib_SOURCE_DIR}")
message(STATUS "PhysX binary directory: ${physx_lib_BINARY_DIR}")

# Don't try to access target properties before they're defined
message(STATUS "PhysX using local checkout at: ${FETCHCONTENT_SOURCE_DIR_PHYSX_LIB}")

# Create an executable for our application
add_executable(hello-physx-cmake main.cpp)

# Link our executable with the physx_lib library
# This now includes all necessary dependencies (PhysX libraries, CUDA, PhysXGpu)
target_link_libraries(hello-physx-cmake PRIVATE physx_lib)

# With our improved physx_lib target, we don't need to manually specify include directories
# All include directories, compile definitions, and other properties are inherited automatically