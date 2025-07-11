cmake_minimum_required(VERSION 3.28.0)
project(yolotest LANGUAGES CXX)
# project(YOLOv8CPP LANGUAGES CXX)

# 设置 C++ 标准
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# --------------------------
# OpenCV 路径配置（基于 tree1.txt）
# --------------------------
set(OpenCV_DIR "D:/OpenCV/opencv/build/x64/vc16/lib")  # OpenCVConfig.cmake 所在目录
find_package(OpenCV REQUIRED)

# --------------------------
# ONNX Runtime 路径配置
# --------------------------
set(ONNXRUNTIME_DIR "D:/YOLOv8/onnxruntime-win-x64-1.18.1")

# 包含头文件目录
include_directories(
    include
    ${OpenCV_INCLUDE_DIRS}
    ${ONNXRUNTIME_DIR}/include
)

# 收集源文件
file(GLOB_RECURSE SOURCES src/*.cpp)

# 创建可执行文件
add_executable(${PROJECT_NAME} ${SOURCES})

# 链接库
target_link_libraries(${PROJECT_NAME}
    ${OpenCV_LIBS}
    ${ONNXRUNTIME_DIR}/lib/onnxruntime.lib
)

# --------------------------
# Windows 平台 DLL 拷贝
# --------------------------
if(WIN32)
    # ONNX Runtime DLL
    add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_if_different
        "${ONNXRUNTIME_DIR}/lib/onnxruntime.dll"
        "$<TARGET_FILE_DIR:${PROJECT_NAME}>"
    )

    # OpenCV DLLs（确保这些 DLL 存在于 bin 目录）
    set(OPENCV_DLL_DIR "D:/OpenCV/opencv/build/x64/vc16/bin")
    set(OPENCV_DLLS
        opencv_world4100.dll
        opencv_videoio_ffmpeg4100_64.dll
    )

    foreach(DLL ${OPENCV_DLLS})
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
            COMMAND ${CMAKE_COMMAND} -E copy_if_different
            "${OPENCV_DLL_DIR}/${DLL}"
            "$<TARGET_FILE_DIR:${PROJECT_NAME}>"
        )
    endforeach()
endif()

# --------------------------
# Linux 平台配置
# --------------------------
if(UNIX)
    target_link_libraries(${PROJECT_NAME}
        ${ONNXRUNTIME_DIR}/lib/libonnxruntime.so
    )
endif()

