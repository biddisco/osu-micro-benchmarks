cmake_minimum_required(VERSION 2.8)
project(osu_microbenchmark)

find_package(MPI REQUIRED)

option(BUILD_MPI3_SUPPORT OFF "compile apps using mpi 3 features")

set(OSU_LIB_SRCS 
  mpi/collective/osu_coll.c
)

set(OSU_PT2PT_LIB_SRCS 
  mpi/pt2pt/osu_pt2pt.c
)

set(OSU_ONE_SIDED_LIB_SRCS 
  mpi/one-sided/osu_1sc.c
)

set(OSU_SRCS
  mpi/startup/osu_hello.c
  mpi/startup/osu_init.c
  mpi/collective/osu_alltoallv.c
  mpi/collective/osu_barrier.c
  mpi/collective/osu_gather.c
  mpi/collective/osu_allreduce.c
  mpi/collective/osu_reduce_scatter.c
  mpi/collective/osu_scatter.c
  mpi/collective/osu_reduce.c
  mpi/collective/osu_bcast.c
  mpi/collective/osu_gatherv.c
  mpi/collective/osu_allgatherv.c
  mpi/collective/osu_alltoall.c
  mpi/collective/osu_scatterv.c
  mpi/collective/osu_allgather.c
  mpi/collective/osu_iallgather.c
  mpi/collective/osu_iallgatherv.c
  mpi/collective/osu_ialltoall.c
  mpi/collective/osu_ialltoallv.c
  mpi/collective/osu_ialltoallw.c
  mpi/collective/osu_ibarrier.c
  mpi/collective/osu_ibcast.c
  mpi/collective/osu_igather.c
  mpi/collective/osu_igatherv.c
  mpi/collective/osu_iscatter.c
  mpi/collective/osu_iscatterv.c
)

set(OSU_PT2PT_SRCS
  mpi/pt2pt/osu_bibw.c
  mpi/pt2pt/osu_mbw_mr.c
  mpi/pt2pt/osu_bw.c
  mpi/pt2pt/osu_latency_mt.c
  mpi/pt2pt/osu_multi_lat.c
  mpi/pt2pt/osu_latency.c
)

# one sided tests
set(OSU_ONE_SIDED_SRCS
  mpi/one-sided/osu_acc_latency.c
  mpi/one-sided/osu_get_bw.c
  mpi/one-sided/osu_get_latency.c
  mpi/one-sided/osu_put_bibw.c
  mpi/one-sided/osu_put_bw.c
  mpi/one-sided/osu_put_latency.c
)

set(OSU_MPI_3_SRCS
  mpi/one-sided/osu_get_acc_latency.c
  mpi/one-sided/osu_fop_latency.c
  mpi/one-sided/osu_cas_latency.c
  mpi/collective/osu_iallgather.c
  mpi/collective/osu_iallgatherv.c
  mpi/collective/osu_ialltoall.c
  mpi/collective/osu_ialltoallv.c
  mpi/collective/osu_ialltoallw.c
  mpi/collective/osu_ibarrier.c
  mpi/collective/osu_ibcast.c
  mpi/collective/osu_igather.c
  mpi/collective/osu_igatherv.c
  mpi/collective/osu_iscatter.c
  mpi/collective/osu_iscatterv.c)

include_directories(
  ${CMAKE_CURRENT_SOURCE_DIR}/mpi/collective
  ${CMAKE_CURRENT_SOURCE_DIR}/mpi/one-sided
  ${CMAKE_CURRENT_SOURCE_DIR}/mpi/pt2pt
  ${MPI_INCLUDE_PATH}
)

add_library(osu_lib ${OSU_LIB_SRCS})
add_library(osu_pt2pt_lib ${OSU_PT2PT_LIB_SRCS})
add_library(osu_one_sided_lib ${OSU_ONE_SIDED_LIB_SRCS})

if(BUILD_MPI3_SUPPORT)
#  set(OSU_ONE_SIDED_SRCS ${OSU_ONE_SIDED_SRCS} ${OSU_MPI_3_SRCS})
#  add_library(osu_one_sided_lib ${OSU_ONE_SIDED_LIB_SRCS})
endif()

function(build_osu_set SRCS LIBS DEFS)
  foreach(file ${SRCS})
    get_filename_component(name_without_extension ${file} NAME_WE)
    message("file is ${file} and name is ${name_without_extension}")
    add_executable(${name_without_extension} ${file})
    target_link_libraries(${name_without_extension} 
      ${LIBS}
      ${MPI_LIBRARY}
      ${MPI_EXTRA_LIBRARY}
      pthread
    )
    if(NOT DEFS STREQUAL "")
      target_compile_definitions(${name_without_extension} PRIVATE ${DEFS})
    endif()
  endforeach()
endfunction()

if(APPLE)
  set(C_DEFS "_POSIX_C_SOURCE=200112L")
  list(REMOVE_ITEM OSU_SRCS mpi/startup/osu_init.c)
endif()
build_osu_set("${OSU_SRCS}" "osu_lib" "${C_DEFS}")
build_osu_set("${OSU_ONE_SIDED_SRCS}" "osu_one_sided_lib" "")
build_osu_set("${OSU_PT2PT_SRCS}" "osu_pt2pt_lib" "FIELD_WIDTH=18;FLOAT_PRECISION=2")
