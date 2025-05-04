# Require minimum CMake version
cmake_minimum_required(VERSION 3.28)

# Define project name and languages
project(hello-physx-cmake LANGUAGES CXX)

#########################################################
# DEPENDENCIES SETUP
#########################################################

# Include required CMake modules
include(FetchContent)       # Module for downloading and configuring external dependencies
include(GNUInstallDirs)     # Sets standard installation directories (bin, lib, etc.)

# PhysX Source Configuration
# If you want to use a local PhysX repository instead of downloading from GitHub,
# set this variable to your local PhysX repository path (without the 'physx' subdirectory)
if(WIN32)
  set(FETCHCONTENT_SOURCE_DIR_PHYSX_LIB "C:/Users/malesiani/PhysX")
else()
  set(FETCHCONTENT_SOURCE_DIR_PHYSX_LIB "/home/alex/fetchcontent_physx/PhysX")
endif()

# Display download progress in the terminal (useful for larger dependencies)
set(FETCHCONTENT_QUIET OFF)

# Declare the PhysX dependency using FetchContent
# - physx_lib: Name used to refer to this dependency in our project
# - GIT_REPOSITORY: Source repository URL
# - GIT_TAG: Specific branch, tag, or commit to use
# - SOURCE_SUBDIR: Subdirectory in the repository containing the main CMakeLists.txt
FetchContent_Declare(
    physx_lib
    GIT_REPOSITORY https://github.com/marcodiiga/PhysX.git
    GIT_TAG        main
    SOURCE_SUBDIR  physx
)

# Set minimum policy version for compatibility
# This ensures consistent behavior with newer CMake versions
set(CMAKE_POLICY_VERSION_MINIMUM 3.5)

# Make PhysX available to our project
# This downloads, configures, and makes PhysX targets available
FetchContent_MakeAvailable(physx_lib)

# Log PhysX directories for debugging purposes
message(STATUS "PhysX source directory: ${physx_lib_SOURCE_DIR}")
message(STATUS "PhysX binary directory: ${physx_lib_BINARY_DIR}")

#########################################################
# APPLICATION SETUP
#########################################################

# Create our application executable
add_executable(hello-physx-cmake main.cpp)

# Link our executable with PhysX
# This automatically includes all necessary dependencies:
# - All PhysX libraries
# - Required include directories
# - Compiler definitions
# - Other properties from the PhysX targets
target_link_libraries(hello-physx-cmake PRIVATE physx_lib)

#########################################################
# INSTALLATION OF EXECUTABLE AND DEPENDENCIES
#########################################################

# Configure runtime path for the executable
# This ensures the executable can find its shared libraries at runtime
set_target_properties(hello-physx-cmake PROPERTIES
  # $ORIGIN refers to the executable's location (Linux specific)
  # This sets the runtime search path to look in ../lib relative to the executable
  INSTALL_RPATH "$ORIGIN/../${CMAKE_INSTALL_LIBDIR}"

  # Don't use install RPATH during the build phase
  BUILD_WITH_INSTALL_RPATH OFF

  # Remove build directory paths from the binary, activate this if you just want
  # to install it and not test it from the build folder
  #  SKIP_BUILD_RPATH        ON
)

if (WIN32)
  # After building hello-physx-cmake, copy all its runtime DLLs next to the EXE.
  # This is useful for windows (since there's no rpath) to find the DLLs at runtime
  add_custom_command(TARGET hello-physx-cmake
    POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy_if_different
      $<TARGET_RUNTIME_DLLS:hello-physx-cmake>
      $<TARGET_FILE_DIR:hello-physx-cmake>
    # ensure the generator expression list is expanded
    COMMAND_EXPAND_LISTS
  )
endif()

# Install our executable and its dependencies
install(TARGETS hello-physx-cmake
  # Configure automatic dependency resolution
  RUNTIME_DEPENDENCIES
    # Exclude system libraries to avoid unnecessary copying
    PRE_EXCLUDE_REGEXES
      [[libc\.so\..*]]          # Standard C library
      [[libstdc\+\+\.so\..*]]   # C++ standard library
      [[.*_static_64\.lib]]      # Static libraries
      [[.*_static\..*]]          # Static libraries (alternative naming)
      [[.*\.lib$]]          # strip out any .lib files
      [[.*\.a$]]   # Exclude all static libraries

    # Specify directories to search for dependencies
    DIRECTORIES
      $<TARGET_FILE_DIR:PhysX>
      $<TARGET_FILE_DIR:PhysXGpu>
      # These are currently always static
      # $<TARGET_FILE_DIR:PhysXCharacterKinematic>
      # $<TARGET_FILE_DIR:PhysXExtensions>
      $<TARGET_FILE_DIR:PhysXCommon>
      $<TARGET_FILE_DIR:PhysXCooking>
      $<TARGET_FILE_DIR:PhysXFoundation>

  # Set installation destinations
  RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}"   # For executables
  LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}"   # For shared libraries
  ARCHIVE DESTINATION ""
)