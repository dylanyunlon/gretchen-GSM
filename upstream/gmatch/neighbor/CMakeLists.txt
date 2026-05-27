cmake_minimum_required(VERSION 3.26)
project(SubgraphMatching LANGUAGES CUDA CXX)

set(CMAKE_VERBOSE_MAKEFILE ON)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -pthread -march=native")

set(CMAKE_CUDA_STANDARD 17)
set(CMAKE_CUDA_STANDARD_REQUIRED ON)

set(CMAKE_CUDA_ARCHITECTURES "80;75")
set(CMAKE_CUDA_FLAGS "${CMAKE_CUDA_FLAGS} -Wno-deprecated-gpu-targets")

find_package(CUDAToolkit REQUIRED)

add_library(graph_objects_unlabeled SHARED graph.cu graph.h)
target_include_directories(graph_objects_unlabeled PUBLIC ${CUDAToolkit_INCLUDE_DIRS})
target_compile_definitions(graph_objects_unlabeled PUBLIC UNLABELED)
target_compile_definitions(graph_objects_unlabeled PRIVATE THRUST_IGNORE_DEPRECATED_CPP_DIALECT)
target_link_libraries(graph_objects_unlabeled PRIVATE CUDA::cudart_static)

add_library(join_objects_unlabeled SHARED
    candidate.cu candidate.h
    join_bfs.cu join_bfs_dfs.cu
    join_no_filtering.cu join_induced.cu join.h)
target_include_directories(join_objects_unlabeled PUBLIC ${CUDAToolkit_INCLUDE_DIRS})
target_compile_definitions(join_objects_unlabeled PUBLIC UNLABELED)
target_compile_definitions(join_objects_unlabeled PRIVATE THRUST_IGNORE_DEPRECATED_CPP_DIALECT)
target_link_libraries(join_objects_unlabeled PRIVATE CUDA::cudart_static)

add_library(graph_objects_labeled SHARED graph.cu graph.h)
target_include_directories(graph_objects_labeled PUBLIC ${CUDAToolkit_INCLUDE_DIRS})
target_compile_definitions(graph_objects_labeled PRIVATE THRUST_IGNORE_DEPRECATED_CPP_DIALECT)
target_link_libraries(graph_objects_labeled PRIVATE CUDA::cudart_static)

add_library(join_objects_labeled SHARED
    candidate.cu candidate.h
    join_bfs.cu join_bfs_dfs.cu
    join_no_filtering.cu join_induced.cu join.h)
target_include_directories(join_objects_labeled PUBLIC ${CUDAToolkit_INCLUDE_DIRS})
target_compile_definitions(join_objects_labeled PRIVATE THRUST_IGNORE_DEPRECATED_CPP_DIALECT)
target_link_libraries(join_objects_labeled PRIVATE CUDA::cudart_static)

set_target_properties(graph_objects_unlabeled graph_objects_labeled
                      join_objects_unlabeled join_objects_labeled PROPERTIES
    CUDA_SEPARABLE_COMPILATION ON
    POSITION_INDEPENDENT_CODE ON)

add_executable(SubgraphMatching_unlabeled main.cu)
target_link_libraries(SubgraphMatching_unlabeled
    PRIVATE graph_objects_unlabeled join_objects_unlabeled CUDA::cudart_static)

add_executable(SubgraphMatching_labeled main.cu)
target_link_libraries(SubgraphMatching_labeled
    PRIVATE graph_objects_labeled join_objects_labeled CUDA::cudart_static)