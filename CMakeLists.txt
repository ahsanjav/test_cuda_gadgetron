cmake_minimum_required(VERSION 3.10...3.12 FATAL_ERROR)

if (${CMAKE_VERSION} VERSION_LESS 3.12)
    cmake_policy(VERSION ${CMAKE_MAJOR_VERSION}.${CMAKE_MINOR_VERSION})
else ()
    cmake_policy(VERSION 3.12)
endif()

set(CMAKE_INSTALL_RPATH ".:${CMAKE_INSTALL_PREFIX}/lib")
set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)


list(APPEND CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake)
set(GADGETRON_INSTALL_INCLUDE_PATH include/gadgetron)

# for non-container efforts..
link_directories("${CMAKE_INSTALL_PREFIX}/lib")

find_package(Gadgetron REQUIRED)
add_definitions(-DARMA_DONT_USE_WRAPPER -DARMA_USE_CXX11 -DARMA_64BIT_WORD)
find_package(CUDA_advanced)
find_package(Boost REQUIRED)
find_package(Python3_compat REQUIRED)
find_package(Armadillo REQUIRED )


set(Boost_USE_STATIC_LIBS OFF)
set(Boost_USE_MULTITHREADED ON)
set(Boost_USE_STATIC_RUNTIME OFF)
# necessary for Windows and RHEL <=6 systems

if(WIN32)
    add_definitions( -DHAVE_SNPRINTF) #Python decides to overwrite snprintf if this is not defined, because ofcourse it does.
endif()
# We actually only use system and thread explicitly, but they require linking in date_time and chrono
if (WIN32)
    find_package(Boost 1.71.0 COMPONENTS coroutine system date_time chrono program_options filesystem timer exception REQUIRED )
else()
    find_package(Boost 1.71.0 COMPONENTS coroutine system date_time program_options filesystem timer REQUIRED )
endif()
add_definitions("-DBOOST_BIND_GLOBAL_PLACEHOLDERS") # To suppress messages in later versions of boost, in version 1.76.0 there are internal conflicts in boost
add_definitions("-DBOOST_ALLOW_DEPRECATED_HEADERS") # To suppress messages in later versions of boost, in version 1.76.0 there are internal conflicts in boost
if (NOT DEFINED Boost_VERSION_STRING)
    set(Boost_VERSION_STRING "${Boost_MAJOR_VERSION}.${Boost_MINOR_VERSION}.${Boost_SUBMINOR_VERSION}")
endif()

find_package(ZFP)


if (ZFP_FOUND)
   add_definitions(-DGADGETRON_COMPRESSION_ZFP)
endif()

if (ZFP_FOUND)
   include_directories(${ZFP_INCLUDE_DIR})
endif()

set( GADGETRON_CUDA_FOUND_BOOL 1 )
include_directories( ${CUDA_INCLUDE_DIRS} )
set(CUDA_PROPAGATE_HOST_FLAGS ON)
set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS} "-std=c++17" )
set(CUDA_HOST_COMPILER ${CMAKE_CXX_COMPILER})


set(CMAKE_CUDA_STANDARD 17)
#set(CURAND_FLAGS "-lcurand_static -lculibos -lcudart_static -lpthread -ldl -I /usr/local/cuda/include -L /usr/local/cuda/lib64")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${CURAND_FLAGS}")

set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS} "-gencode arch=compute_60,code=sm_60")
set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS} "-gencode arch=compute_61,code=sm_61")
set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS} "-gencode arch=compute_70,code=sm_70")
set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS} "-gencode arch=compute_75,code=sm_75")
set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS} "-gencode arch=compute_80,code=sm_80")

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_executable(test_cuda_gadgetron test_cuda_gadgetron.cpp)

target_link_libraries(test_cuda_gadgetron gadgetron_core gadgetron_toolbox_cpucore_math gadgetron_toolbox_spiral gadgetron_toolbox_cpunfft)

target_link_libraries(test_cuda_gadgetron
  gadgetron_toolbox_hostutils 
  gadgetron_toolbox_mri_core
  gadgetron_toolbox_cpusdc
  gadgetron_mri_noncartesian
  gadgetron_toolbox_spiral 
  gadgetron_toolbox_cpunfft
  gadgetron_toolbox_log
  ${CUDA_LIBRARIES}
  Boost::program_options
  gadgetron_toolbox_gpu
 )

#  target_link_libraries(test_cuda_gadgetron
#  gadgetron_toolbox_hostutils 
#  gadgetron_toolbox_mri_core
#  gadgetron_toolbox_cpusdc
#  gadgetron_mri_noncartesian
#  gadgetron_toolbox_spiral 
#  gadgetron_toolbox_cpunfft
#  gadgetron_toolbox_log
#  ${CUDA_LIBRARIES}
#  Boost::program_options
#  gadgetron_toolbox_gpucore
#  gadgetron_toolbox_gpusolvers
#  gadgetron_toolbox_gpuoperators
#  gadgetron_toolbox_gpuparallelmri
#  gadgetron_toolbox_gpunfft
#	  )
  
target_include_directories(test_cuda_gadgetron
        PUBLIC    
	$<INSTALL_INTERFACE:include>
    ${CONDA_PREFIX}/include
)

install(TARGETS test_cuda_gadgetron DESTINATION bin COMPONENT main)

