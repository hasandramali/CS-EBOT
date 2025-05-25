cmake_minimum_required(VERSION 3.16)
project(EBOT)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_BUILD_TYPE Release)

# Set output directory
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/build/ebot)

# Determine target platform
if(WIN32)
    set(BUILDING_ON_WINDOWS TRUE)
    set(CMAKE_SHARED_LIBRARY_SUFFIX ".dll")
else()
    set(BUILDING_ON_WINDOWS FALSE)
    set(CMAKE_SHARED_LIBRARY_SUFFIX ".so")
endif()

# Force 32-bit compilation
if(MSVC)
    message(STATUS "MSVC detected, assume environment is set for 32-bit build.")
elseif(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -m32")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -m32")
elseif(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -m32")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -m32")
endif()

# Set compiler flags per platform and build type
if(BUILDING_ON_WINDOWS)
    if(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -m32 -DWIN32_LEAN_AND_MEAN")
    endif()
else()
    if(CMAKE_BUILD_TYPE STREQUAL "Debug")
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -g3 -Og")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -g3 -Og -fno-inline -fno-omit-frame-pointer -fno-threadsafe-statics -pthread")
    else()
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Ofast")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -w -mmmx -msse -msse2 -mfpmath=sse -flto -Ofast -fvisibility=hidden -fno-exceptions -fno-rtti -pipe -mtune=generic -fomit-frame-pointer -fvisibility-inlines-hidden -fno-stack-protector -no-pie -fno-threadsafe-statics -pthread")
        set(CMAKE_CXX_LINK_FLAGS "${CMAKE_CXX_LINK_FLAGS} -m32")
        set(CMAKE_INTERPROCEDURAL_OPTIMIZATION TRUE)
    endif()
endif()

# Include & source
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/../include)
file(GLOB_RECURSE EBOT_SRC ${CMAKE_CURRENT_SOURCE_DIR}/../source/*.cpp)

# Build shared library
add_library(${PROJECT_NAME} SHARED ${EBOT_SRC})

# Link libraries
if(BUILDING_ON_WINDOWS)
    target_link_libraries(${PROJECT_NAME} PRIVATE ws2_32)
else()
    find_package(Threads REQUIRED)
    target_link_libraries(${PROJECT_NAME} PRIVATE m dl pthread)
endif()

# Target properties
set_target_properties(${PROJECT_NAME} PROPERTIES
    CXX_STANDARD 11
    CXX_STANDARD_REQUIRED ON
    POSITION_INDEPENDENT_CODE ON
    PREFIX ""
    OUTPUT_NAME "ebot"
    INTERPROCEDURAL_OPTIMIZATION ON
)

# Install
install(TARGETS ${PROJECT_NAME} LIBRARY DESTINATION lib)
install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/../include/ DESTINATION include)
