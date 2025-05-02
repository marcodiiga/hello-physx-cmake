cmake_minimum_required(VERSION 3.28)
project(hello-physx-cmake LANGUAGES CXX)

# Include the FetchContent module
include(FetchContent)
include(GNUInstallDirs) # sets any install default dir correctly

# DEBUG - if you want to use a local git repository instead of the remote github one, set this to your local checkout path:
# WARNING: do NOT include the 'physx' subdirectory, just the root of the repository!!!
set(FETCHCONTENT_SOURCE_DIR_PHYSX_LIB "/home/alex/fetchcontent_physx/PhysX")

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

# set(PHYSX_AS_SUBPROJECT ON)
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

# 1. Retrieve the two binary output paths from the target’s INTERFACE_ properties
get_target_property(_PHYSX_SRC_BIN physx_lib INTERFACE_PHYSX_SDK_SOURCE_BIN_DIR)
get_target_property(_PHYSX_GPU_BIN physx_lib INTERFACE_PHYSX_SDK_GPU_SOURCE_BIN_DIR)

message(STATUS "PhysX core binaries in: ${_PHYSX_SRC_BIN}")
message(STATUS "PhysX GPU binaries in: ${_PHYSX_GPU_BIN}")

# Install executable with automatic dependency resolution
# Also Automatically DETECT (modern cmake) and install through physx_lib ALL the PhysX
# BINARY dependencies (not headers here!!!) and dependent libraries.
# If I wanted to install the headers, I would have used find_package(PhysX REQUIRED)
# and then used the PhysX::PhysX target.
# Install all detected runtime dependencies (shared libraries)
# TODO

# at the very top, or right before your install() call
message(STATUS "CMake version: ${CMAKE_VERSION}")
message(STATUS "Major: ${CMAKE_MAJOR_VERSION}, Minor: ${CMAKE_MINOR_VERSION}, Patch: ${CMAKE_PATCH_VERSION}")

# Tell the installed EXE to look for its .so files in ../lib
# relative to itself.
set_target_properties(hello-physx-cmake PROPERTIES
  INSTALL_RPATH "$ORIGIN/../${CMAKE_INSTALL_LIBDIR}"
  BUILD_WITH_INSTALL_RPATH OFF    # don't use install RPATH during build
  SKIP_BUILD_RPATH        ON     # strip RPATH from the build‐tree EXE
)

install(TARGETS hello-physx-cmake
  RUNTIME_DEPENDENCIES
    PRE_EXCLUDE_REGEXES
      [[libc\.so\..*]]
      [[libstdc\+\+\.so\..*]]
    DIRECTORIES
      $<TARGET_FILE_DIR:PhysX>
      $<TARGET_FILE_DIR:PhysXGpu>
      $<TARGET_FILE_DIR:PhysXCharacterKinematic>
      $<TARGET_FILE_DIR:PhysXCommon>
      $<TARGET_FILE_DIR:PhysXCooking>
      $<TARGET_FILE_DIR:PhysXExtensions>
      $<TARGET_FILE_DIR:PhysXFoundation>
  RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}"
  LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}"
)