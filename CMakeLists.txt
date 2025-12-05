cmake_minimum_required(VERSION 2.8)   # <- put this BEFORE project() to avoid the warning

# define the project
project(indi_starpoint_focuser C CXX)

include(GNUInstallDirs)

# add cmake_modules folder
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake_modules/")
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/../cmake_modules/")

# find our required packages
find_package(Nova REQUIRED)
find_package(ZLIB REQUIRED)
find_package(GSL REQUIRED)

find_path(INDI_INCLUDE_DIR
    NAMES indidevapi.h indiversion.h
    PATHS /usr/include/libindi /usr/local/include/libindi
)

find_library(INDI_DRIVER_LIB
    NAMES indidriver
    PATHS /usr/lib /usr/lib/x86_64-linux-gnu /usr/local/lib
)

# Convenience variables
set(INDI_INCLUDE_DIRS ${INDI_INCLUDE_DIR})
set(INDI_DATA_DIR     share/indi) 

# set the version number in config.h and driver's xml file
set(CDRIVER_VERSION_MAJOR 1)
set(CDRIVER_VERSION_MINOR 2)

# do the replacement in the config.h
configure_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/config.h.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/config.h
)

# do the replacement in the driver's xml file
configure_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/indi_starpoint_focuser.xml.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/indi_starpoint_focuser.xml
)

# set our include directories to look for header files
include_directories(${CMAKE_CURRENT_BINARY_DIR})
include_directories(${CMAKE_CURRENT_SOURCE_DIR})
include_directories(${INDI_INCLUDE_DIR})
include_directories(${NOVA_INCLUDE_DIR})

include(CMakeCommon)

# make sure we override any INDI libs set in CMakeCommon
set(INDI_LIBRARIES ${INDI_DRIVER_LIB})

# tell cmake to build our executable
add_executable(
    indi_starpoint_focuser
    indi_starpoint_focuser.cpp
)

# link to libraries
target_link_libraries(
    indi_starpoint_focuser
    ${INDI_LIBRARIES}
    ${NOVA_LIBRARIES}
    ${GSL_LIBRARIES}
)

# tell cmake where to install our executable
install(TARGETS indi_starpoint_focuser RUNTIME DESTINATION bin)

# and where to put the driver's xml file.
install(
    FILES
        ${CMAKE_CURRENT_BINARY_DIR}/indi_starpoint_focuser.xml
    DESTINATION
        ${INDI_DATA_DIR}
)
