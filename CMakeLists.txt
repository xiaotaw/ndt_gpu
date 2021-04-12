cmake_minimum_required(VERSION 2.8.3)
project(ndt_gpu)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -O3 -Wall -mtune=native")

set(EIGEN_ROOT "/home/xt/Documents/data/Others/eigen-3.3.9")
set(EIGEN_INCLUDE_DIR ${EIGEN_ROOT})
INCLUDE_DIRECTORIES(${EIGEN_INCLUDE_DIR})

find_package(PCL REQUIRED)

find_package(Boost)

find_package(CUDA)
if (CUDA_FOUND)
	set(CUDA_NVCC_FLAGS "${CUDA_NVCC_FLAGS} -O3 -std=c++11 -gencode=arch=compute_61,code=sm_61")
	
	set(SUBSYS_NAME ndt_gpu)
	set(SUBSYS_DESC "Point cloud ndt gpu library")
	set(LIB_NAME "fast_pcl_ndt_gpu")
	
    include_directories(
      ${CUDA_INCLUDE_DIRS}
      ${PCL_INCLUDE_DIRS}
      "${CMAKE_CURRENT_SOURCE_DIR}/include"
    )

	set(srcs
	  src/MatrixDevice.cu
		src/MatrixHost.cu
		src/NormalDistributionsTransform.cu
		src/Registration.cu
		src/VoxelGrid.cu
		src/SymmetricEigenSolver.cu
	)
	
	set(incs
	  include/fast_pcl/ndt_gpu/common.h
	  include/fast_pcl/ndt_gpu/debug.h
	  include/fast_pcl/ndt_gpu/Matrix.h
	  include/fast_pcl/ndt_gpu/MatrixDevice.h
	  include/fast_pcl/ndt_gpu/MatrixHost.h
	  include/fast_pcl/ndt_gpu/NormalDistributionsTransform.h
	  include/fast_pcl/ndt_gpu/Registration.h
	  include/fast_pcl/ndt_gpu/SymmetricEigenSolver.h
	  include/fast_pcl/ndt_gpu/VoxelGrid.h
	)
	
	cuda_add_library("${LIB_NAME}" ${srcs} ${incs})
	
	target_include_directories("${LIB_NAME}" PRIVATE
		${CUDA_INCLUDE_DIRS}
	)
	
	target_link_libraries("${LIB_NAME}" 
		${CUDA_LIBRARIES}
		${CUDA_CUBLAS_LIBRARIES}
		${CUDA_curand_LIBRARY})


	add_executable(test_node src/test_node.cpp)	
	#target_include_directories(test_node PRIVATE ${CUDA_INCLUDE_DIRS})
	target_link_libraries(test_node
		${LIB_NAME}
		#${catkin_LIBRARIES} 
  	    ${PCL_LIBRARIES}
        ${Boost_LIBRARIES}
	)
else()
	message("fast_pcl ndt_gpu requires CUDA")
endif()
