cmake_minimum_required(VERSION 3.31)
set (CMAKE_CXX_STANDARD 17)
project(shape_based_matching)
# 导出编译命令供 VS Code IntelliSense 使用
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)


# debug or release
SET(CMAKE_BUILD_TYPE "Release")


# arm or x86
IF(${CMAKE_SYSTEM_PROCESSOR} MATCHES "arm")
    SET(PLATFORM_COMPILE_FLAGS "-mfpu=neon")
ELSE()
    SET(PLATFORM_COMPILE_FLAGS "-march=native")

    # some places of the algorithm are designed for 128 SIMD
    # so 128 SSE may slightly faster than 256 AVX, you may want this
#    SET(PLATFORM_COMPILE_FLAGS "-msse -msse2 -msse3 -msse4 -mssse3")  # SSE only
ENDIF()

# SET(PLATFORM_COMPILE_FLAGS "-DMIPP_NO_INTRINSICS")  # close SIMD

# macOS clang 不支持 -fopenmp，检查并条件添加
IF(APPLE)
    SET(COMMON_COMPILE_FLAGS "-Wall -Wno-sign-compare")
ELSE()
    SET(COMMON_COMPILE_FLAGS "-fopenmp -Wall -Wno-sign-compare")
ENDIF()

SET(CMAKE_CXX_FLAGS "${PLATFORM_COMPILE_FLAGS} ${COMMON_COMPILE_FLAGS} $ENV{CXXFLAGS}")
SET(CMAKE_CXX_FLAGS_DEBUG "-O0 -g2 -ggdb")
SET(CMAKE_CXX_FLAGS_RELEASE "-O3")

set(OpenCV_DIR "/opt/local/libexec/opencv4/cmake")

# 显式设置 OpenCV 头文件搜索路径
set(OpenCV_INCLUDE_DIRS 
    "/opt/local/include/opencv4"
    "/opt/local/include/opencv4/opencv2"
)

# 显式设置 OpenCV 库文件搜索路径
set(OpenCV_LIBRARY_DIRS
    "/opt/local/lib/opencv4"
    "/opt/local/lib"
)

# 查找 OpenCV 包
find_package(OpenCV 4 REQUIRED)

# 显示找到的 OpenCV 信息
message(STATUS "OpenCV library status:")
message(STATUS "    version: ${OpenCV_VERSION}")
message(STATUS "    libraries: ${OpenCV_LIBS}")
message(STATUS "    include path: ${OpenCV_INCLUDE_DIRS}")
message(STATUS "    library path: ${OpenCV_LIBRARY_DIRS}")

# include MIPP headers
include_directories (${INCLUDE_DIRECTORIES} "${CMAKE_CURRENT_SOURCE_DIR}/MIPP/")

# test exe
add_executable(${PROJECT_NAME}_test line2Dup.cpp test.cpp)

# 包含头文件目录（必须在 add_executable 之后）
target_include_directories(${PROJECT_NAME}_test PRIVATE 
    ${OpenCV_INCLUDE_DIRS}
)

target_link_libraries(${PROJECT_NAME}_test ${OpenCV_LIBS})

