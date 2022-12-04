cmake_minimum_required(VERSION 3.23)


######## Project information ########
set(Project_Title       EyeTracker)
set(Project_Version     0.0.0)
set(Project_Author      ClaraCF)
#####################################


# Configure project
project(${Project_Title} LANGUAGES CXX VERSION ${Project_Version})
set(CMAKE_CXX_STANDARD 17 REQUIRED ON)


### Options ###

## OPTION: ENABLE_LTO
# Enable Link Time Optimizations (LTO) on Release build type. On by Default.
# This MAY cause problems on some systems.
# If it does, it can be disabled with -DENABLE_LTO=OFF.
option(ENABLE_LTO "Enable Link Time Optimizations" ON)

## OPTION: USE_SYSTEM_OPENCV_LIB
# Use the library bundled with the system for OpenCV instead of compiling it
# NOT recommended, as when compiled in this project, optimizations specific
# for your system are applied. Off by Default.
option(USE_SYSTEM_OPENCV_LIB "Use the system OpenCV library" OFF)

## OPTION: USE_SYSTEM_DLIB_LIB
# Use the library bundled with the system for Dlib instead of compiling it
# NOT recommended, as when compiled in this project, optimizations specific
# for your system are applied. Off by Default.
option(USE_SYSTEM_DLIB_LIB "Use the system Dlib library" OFF)


# Set the CMake Build Type to Release by default
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release)
endif()


### Libraries ###

# PkgConfig to find libraries on Unix systems
if(NOT WIN32)
    # This will throw an error and exit if PkgConfig
    # is not installed in the user's computer
    find_package(PkgConfig REQUIRED)
endif()


# OpenCV Library
if(NOT USE_SYSTEM_OPENCV_LIB)
    # Add OpenCV library from source
    add_subdirectory(lib/opencv)
else()
    # Try to find the OpenCV library on the system
    # NOTE: If the library is not found, falling
    # back to the local library will NOT be attempted
    # and it is needed to either reconfigure the project
    # without USE_SYSTEM_OPENCV_LIB or
    # installing the library on your system
    find_package(OpenCV REQUIRED)
endif()

# Dlib Library
if(NOT USE_SYSTEM_DLIB_LIB)
    # Add Dlib library from source
    add_subdirectory(lib/dlib dlib)

    # Set Dlib variables
    set(DLIB_INCLUDE_DIRS "lib/dlib/")
    set(DLIB_LIBS "dlib::dlib")
else()
    # Try to find the Dlib library on the system
    # NOTE: If the library is not found, falling
    # back to the local library will NOT be attempted
    # and it is needed to either reconfigure the project
    # without USE_SYSTEM_DLIB_LIB or
    # installing the library on your system
    find_package(Dlib REQUIRED)
endif()


# Define some flags for increased performance
# NOTE: -pipe does not affect the final binary
# It only speeds up compilation by avoiding writing temp files
set(SYSTEM_TUNING_FLAGS "-march=native -mtune=native -pipe -O3")

# Add LTO flags if enabled
if(ENABLE_LTO)
    set(SYSTEM_TUNING_FLAGS "${SYSTEM_TUNING_FLAGS} -flto")
endif()

# Add additional flags for performance to Release build type
if(CMAKE_BUILD_TYPE "Release")
    target_compile_options(${PROJECT_NAME} PRIVATE "${SYSTEM_TUNING_FLAGS}")
    target_link_libraries(${PROJECT_NAME} PRIVATE "${SYSTEM_TUNING_FLAGS} -s")
endif()



### Project Sources ###
set(PROJECT_HEADERS
)

set(PROJECT_SOURCES
    src/main.cpp
)


# Target binary
add_executable(${PROJECT_NAME} ${PROJECT_HEADERS} ${PROJECT_SOURCES})

# Add a version string
target_compile_definitions(${PROJECT_NAME} "${PROJECT_VERSION}-alpha")  # Alpha

# Project include directories
target_include_directories(${PROJECT_NAME} PUBLIC include/)


# Specify include directories for libraries
target_include_directories(${PROJECT_NAME}
    PUBLIC ${OpenCV_INCLUDE_DIRS}
    PUBLIC ${DLIB_INCLUDE_DIRS}
)

# Link libraries
target_link_libraries(${PROJECT_NAME}
    PUBLIC OPTIMIZED ${OpenCV_LIBS}
    PUBLIC OPTIMIZED ${DLIB_LIBS}
)
