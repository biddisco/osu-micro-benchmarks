cmake_minimum_required(VERSION 2.8)
project(osu_microbenchmark)

find_package(MPI REQUIRED)

option(BUILD_MPI3_SUPPORT OFF "compile apps using mpi 3 features")

set(OSU_LIB_SRCS 
  ./mpi/collective/osu_coll.c
)

set(OSU_SRCS
  ./mpi/collective/osu_alltoallv.c
  ./mpi/collective/osu_barrier.c
  ./mpi/collective/osu_gather.c
  ./mpi/collective/osu_allreduce.c
  ./mpi/collective/osu_reduce_scatter.c
  ./mpi/collective/osu_scatter.c
  ./mpi/collective/osu_reduce.c
  ./mpi/collective/osu_bcast.c
  ./mpi/collective/osu_gatherv.c
  ./mpi/collective/osu_allgatherv.c
  ./mpi/collective/osu_alltoall.c
  ./mpi/collective/osu_scatterv.c
  ./mpi/collective/osu_allgather.c
  ./mpi/pt2pt/osu_bibw.c
  ./mpi/pt2pt/osu_mbw_mr.c
  ./mpi/pt2pt/osu_bw.c
  ./mpi/pt2pt/osu_latency_mt.c
  ./mpi/pt2pt/osu_multi_lat.c
  ./mpi/pt2pt/osu_latency.c
  ./mpi/one-sided/osu_get_latency.c
  ./mpi/one-sided/osu_put_latency.c
  ./mpi/one-sided/osu_acc_latency.c
  ./mpi/one-sided/osu_put_bibw.c
)

set(OSU_SRCS_3
  ./mpi/one-sided/osu_get_acc_latency.c
  ./mpi/one-sided/osu_fop_latency.c
  ./mpi/one-sided/osu_cas_latency.c
  ./mpi/one-sided/osu_get_bw.c
  ./mpi/one-sided/osu_put_bw.c
)

if(BUILD_MPI3_SUPPORT)
  set(OSU_SRCS ${OSU_SRCS} ${OSU_SRCS_3})
endif()

include_directories(
  ${CMAKE_CURRENT_SOURCE_DIR}/mpi/collective
  ${MPI_INCLUDE_PATH}
)

add_library(osu_lib ${OSU_LIB_SRCS})

foreach(file ${OSU_SRCS})
  get_filename_component(name_without_extension ${file} NAME_WE)
  message("file is ${file} and name is ${name_without_extension}")
  add_executable(${name_without_extension} ${file})
  target_link_libraries(${name_without_extension} 
    osu_lib
    ${MPI_LIBRARY}
    ${MPI_EXTRA_LIBRARY}
    pthread
  )
endforeach()

