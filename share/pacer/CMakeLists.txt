cmake_minimum_required(VERSION 3.18)

# Needs to happen before project call
function(set_compilers_e3sm)
  # Grab CXX compiler from CIME
  include(${CASEROOT}/Macros.cmake)

  set(CMAKE_CXX_COMPILER     ${CMAKE_CXX_COMPILER}     CACHE STRING "The CXX compiler")
endfunction()

set_compilers_e3sm()

include(${CASEROOT}/Macros.cmake)

project(PACER CXX)

set(CMAKE_VERBOSE_MAKEFILE TRUE)

include(${CASEROOT}/Macros.cmake)

set(SRCS_CXX
  src/Pacer.cpp
  )

set(PACER_INCLUDE_DIRS
  ${CMAKE_CURRENT_SOURCE_DIR}
  ${CMAKE_CURRENT_BINARY_DIR}
  ${PACER_DIR}/include
  ${GPTL_DIR}/include
  ${INSTALL_SHAREDPATH}/include)

if (NOT MPILIB STREQUAL "mpi-serial")
  string(APPEND PACER_CPPDEFS " -DHAVE_MPI")
endif()
if (compile_threaded)
  string(APPEND PACER_CPPDEFS " -DTHREADED_OMP")
endif()
if (DEBUG)
  string(APPEND PACER_CPPDEFS " -DDEBUG")
endif()

set(CPPDEFS "${CPPDEFS} ${PACER_CPPDEFS}")
separate_arguments(CPPDEFS_LIST UNIX_COMMAND "${CPPDEFS}")

add_library(pacer ${SRCS_CXX})
target_include_directories(pacer PRIVATE ${PACER_INCLUDE_DIRS})
target_compile_definitions(pacer PRIVATE ${CPPDEFS_LIST})
set_target_properties(pacer PROPERTIES PUBLIC_HEADER "Pacer.h;")

install(TARGETS pacer
  PUBLIC_HEADER DESTINATION include)
