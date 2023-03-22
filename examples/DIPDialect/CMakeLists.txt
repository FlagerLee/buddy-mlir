if(BUDDY_ENABLE_OPENCV)
  find_package(OpenCV REQUIRED CONFIG)
  include_directories(${OpenCV_INCLUDE_DIRS})
endif()

if(OPENMP_LIB)
  set(CMAKE_CXX_COMPILER ${LLVM_MLIR_BINARY_DIR}/clang++)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -no-pie")
  link_libraries(${OPENMP_LIB})
endif()

add_executable(correlation2D correlation2D.cpp)
target_link_libraries(correlation2D ${OpenCV_LIBS} BuddyLibDIP)

add_executable(rotation2D rotation2D.cpp)
target_link_libraries(rotation2D ${OpenCV_LIBS} BuddyLibDIP)

add_executable(resize2D resize2D.cpp)
target_link_libraries(resize2D ${OpenCV_LIBS} BuddyLibDIP)

add_executable(morph2D morph2D.cpp)
target_link_libraries(morph2D ${OpenCV_LIBS} BuddyLibDIP)

if(NOT OPENMP_LIB)
  target_link_libraries(correlation2D static_mlir_async_runtime)
  target_link_libraries(rotation2D static_mlir_async_runtime)
  target_link_libraries(resize2D static_mlir_async_runtime)
  target_link_libraries(morph2D static_mlir_async_runtime)
endif()