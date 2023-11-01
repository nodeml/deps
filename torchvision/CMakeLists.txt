cmake_minimum_required(VERSION 3.18 FATAL_ERROR)

include(${CMAKE_SOURCE_DIR}/utils.cmake)

set(CMAKE_CXX_STANDARD 17)

find_package(PNG REQUIRED)
find_package(JPEG REQUIRED)

option(WITH_CUDA "Build with cuda?" OFF)

set(PYTORCH_VERSION "2.1.0" CACHE STRING "Torch Version")

set(PYTORCH_VISION_VERSION "0.16.0" CACHE STRING "Torchvision Version" )

set(TORCH_DEPS_DIR "${CMAKE_SOURCE_DIR}/deps/libtorch")
list(APPEND CMAKE_PREFIX_PATH ${TORCH_DEPS_DIR})
list(APPEND CMAKE_PREFIX_PATH "${TORCH_DEPS_DIR}/share/cmake/Torch")

DownloadTorch(${PYTORCH_VERSION} ${WITH_CUDA} "${CMAKE_SOURCE_DIR}/deps/")

find_package(Torch REQUIRED)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${TORCH_CXX_FLAGS}")

# Torchvision
set(TORCH_VISION_DEPS_DIR ${CMAKE_SOURCE_DIR}/torchvision)
list(APPEND CMAKE_PREFIX_PATH ${TORCH_VISION_DEPS_DIR})
list(APPEND CMAKE_PREFIX_PATH "${TORCH_VISION_DEPS_DIR}/share/cmake/TorchVision")

execute_process(
  COMMAND git clone --depth 1 --branch v${PYTORCH_VISION_VERSION} https://github.com/pytorch/vision "${CMAKE_CURRENT_BINARY_DIR}/cloned/"
)

status(STATUS "TORCH DEPS DIR ${TORCH_DEPS_DIR}")
execute_process(
  COMMAND ${CMAKE_COMMAND} -DCMAKE_BUILD_TYPE=Release -DUSE_PYTHON=OFF -DWITH_CUDA=${WITH_CUDA} -DWITH_PNG=ON -DWITH_JPEG=ON -DCMAKE_PREFIX_PATH=${TORCH_DEPS_DIR} -S "${CMAKE_CURRENT_BINARY_DIR}/cloned/" -B "${CMAKE_CURRENT_BINARY_DIR}/cloned/build/"
)

execute_process(
  COMMAND ${CMAKE_COMMAND} --build "${CMAKE_CURRENT_BINARY_DIR}/cloned/build" --config Release
)

execute_process(
  COMMAND ${CMAKE_COMMAND} --install "${CMAKE_CURRENT_BINARY_DIR}/cloned/build" --prefix ${CMAKE_SOURCE_DIR}/torchvision
)



if (MSVC)
  file(GLOB PACKAGE_DLLS  ${PNG_LIBRARIES}  ${JPEG_LIBRARIES})
  file(COPY ${PACKAGE_DLLS} DESTINATION "${TORCH_VISION_DEPS_DIR}/bin")
  # add_custom_command(TARGET ${PROJECT_NAME}
  #                    POST_BUILD
  #                    COMMAND ${CMAKE_COMMAND} -E copy_if_different
  #                    ${PACKAGE_DLLS}
  #                    )
  
endif (MSVC)