cmake_minimum_required(VERSION 3.18 FATAL_ERROR)

set(CMAKE_CXX_STANDARD 17)

# option(WITH_CUDA "Enable CUDA support" OFF)

set(OPENCV_VERSION "4.8.1" CACHE STRING "OpenCV Version")
message(STATUS "VERSION ${OPENCV_VERSION}")
execute_process(
  COMMAND git clone --depth 1 --branch ${OPENCV_VERSION} https://github.com/opencv/opencv cloned
)

execute_process(
  COMMAND ${CMAKE_COMMAND} -DCMAKE_BUILD_TYPE=Release -DWITH_TBB=ON -DWITH_OPENMP=ON -DWITH_IPP=ON -DBUILD_EXAMPLES=OFF -DBUILD_DOCS=OFF -DBUILD_PERF_TESTS=OFF -DBUILD_TESTS=OFF -DWITH_CSTRIPES=ON -DWITH_OPENCL=ON -S "${CMAKE_CURRENT_BINARY_DIR}/cloned/" -B "${CMAKE_CURRENT_BINARY_DIR}/cloned/build/"
)

execute_process(
  COMMAND ${CMAKE_COMMAND} --build "${CMAKE_CURRENT_BINARY_DIR}/cloned/build" --config Release
)

execute_process(
  COMMAND ${CMAKE_COMMAND} --install "${CMAKE_CURRENT_BINARY_DIR}/cloned/build" --prefix ${CMAKE_SOURCE_DIR}/opencv
)