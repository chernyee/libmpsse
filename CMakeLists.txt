cmake_minimum_required(VERSION 3.10)

project(libmpsse C)

set(CMAKE_C_STANDARD 99)
set(CMAKE_C_STANDARD_REQUIRED ON)

find_package(PkgConfig REQUIRED)
pkg_check_modules(FTDI REQUIRED libftdi1)

if(NOT FTDI_FOUND)
    message(FATAL_ERROR "libftdi1 not found. Please install it (e.g., `sudo apt install libftdi1-dev`)")
endif()

file(GLOB LIB_SOURCES "${CMAKE_CURRENT_SOURCE_DIR}/src/*.c")

# Custom command to run ./configure
add_custom_command(
    OUTPUT ${CMAKE_CURRENT_SOURCE_DIR}/src/config.h
    COMMAND ./configure --disable-python
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/src
    COMMENT "Running ./configure for libmpsse"
    VERBATIM
)

# Target to run ./configure
add_custom_target(
    configure_libmpsse
    DEPENDS DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/src/config.h
)

add_library(mpsse STATIC ${LIB_SOURCES})
set_target_properties(mpsse PROPERTIES POSITION_INDEPENDENT_CODE ON)
target_compile_definitions(mpsse PRIVATE LIBFTDI1=1)
target_link_libraries(mpsse PRIVATE ${FTDI_LIBRARIES})
add_dependencies(mpsse configure_libmpsse)

# Optionally, specify public headers
target_include_directories(mpsse PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/src)
