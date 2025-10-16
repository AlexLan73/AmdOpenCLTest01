cmake_minimum_required(VERSION 3.16)
project(UnifiedFFT VERSION 1.0.0 LANGUAGES CXX CUDA)

# Set C++ standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Set CUDA standard
set(CMAKE_CUDA_STANDARD 17)
set(CMAKE_CUDA_STANDARD_REQUIRED ON)

# Enable all warnings
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -Wpedantic")
set(CMAKE_CUDA_FLAGS "${CMAKE_CUDA_FLAGS} -Xcompiler -Wall,-Wextra,-Wpedantic")

# Build type
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release)
endif()

# Optimizations
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -O3 -DNDEBUG")
set(CMAKE_CUDA_FLAGS_RELEASE "${CMAKE_CUDA_FLAGS_RELEASE} -O3 -DNDEBUG")

# Include directories
include_directories(${CMAKE_SOURCE_DIR}/Shared)
include_directories(${CMAKE_SOURCE_DIR}/NVIDIA/Custom/include)
include_directories(${CMAKE_SOURCE_DIR}/OpenCL/Custom/include)
include_directories(${CMAKE_SOURCE_DIR}/Vulkan/Custom/include)

# Find packages
find_package(CUDA REQUIRED)
find_package(OpenCL REQUIRED)
find_package(Vulkan REQUIRED)

# Shared components
add_subdirectory(Shared)

# Platform-specific components
add_subdirectory(NVIDIA)
add_subdirectory(OpenCL)
add_subdirectory(Vulkan)

# Main application
add_executable(unified_fft_app main.cpp)
target_link_libraries(unified_fft_app 
    UnifiedFFT_Shared
    UnifiedFFT_NVIDIA
    UnifiedFFT_OpenCL
    UnifiedFFT_Vulkan
    ${CUDA_LIBRARIES}
    ${OpenCL_LIBRARIES}
    ${Vulkan_LIBRARIES}
)

# Test demo
add_executable(test_demo test_demo.cpp)
target_link_libraries(test_demo
    UnifiedFFT_Shared
    UnifiedFFT_NVIDIA
    UnifiedFFT_OpenCL
    UnifiedFFT_Vulkan
    ${CUDA_LIBRARIES}
    ${OpenCL_LIBRARIES}
    ${Vulkan_LIBRARIES}
)

# Installation
install(TARGETS unified_fft_app test_demo
    RUNTIME DESTINATION bin
)

install(DIRECTORY Results/
    DESTINATION share/UnifiedFFT/results
)