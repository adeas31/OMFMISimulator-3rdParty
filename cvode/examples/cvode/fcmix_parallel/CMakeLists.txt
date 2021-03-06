# ---------------------------------------------------------------
# $Revision: 4957 $
# $Date: 2016-09-23 12:21:47 -0700 (Fri, 23 Sep 2016) $
# ---------------------------------------------------------------
# Programmer:  Radu Serban @ LLNL
# ---------------------------------------------------------------
# LLNS Copyright Start
# Copyright (c) 2014, Lawrence Livermore National Security
# This work was performed under the auspices of the U.S. Department 
# of Energy by Lawrence Livermore National Laboratory in part under 
# Contract W-7405-Eng-48 and in part under Contract DE-AC52-07NA27344.
# Produced at the Lawrence Livermore National Laboratory.
# All rights reserved.
# For details, see the LICENSE file.
# LLNS Copyright End
# ---------------------------------------------------------------
# CMakeLists.txt file for the FCVODE parallel examples


# Add variable FCVODE_examples with the names of the parallel CVODE examples

SET(FCVODE_examples
  "fcvDiag_non_p\;1\;2"
  "fcvDiag_kry_bbd_p\;1\;2"
  "fcvDiag_kry_p\;1\;2"
  )

# Check whether we use MPI compiler scripts.
# If yes, then change the Fortran compiler to the MPIF77 script.
# If not, then add the MPI include directory for MPI headers.

IF(MPI_MPIF77 )
  # use MPI_MPIF77 as the compiler
  SET(CMAKE_Fortran_COMPILER ${MPI_MPIF77})
ELSE(MPI_MPIF77)
  # add MPI_INCLUDE_PATH to include directories
  INCLUDE_DIRECTORIES(${MPI_INCLUDE_PATH})
ENDIF(MPI_MPIF77)

# Specify libraries to link against (through the target that was used to 
# generate them) based on the value of the variable LINK_LIBRARY_TYPE

IF(LINK_LIBRARY_TYPE MATCHES "static")
  SET(CVODE_LIB sundials_cvode_static)
  SET(NVECP_LIB sundials_nvecparallel_static)
  SET(FNVECP_LIB sundials_fnvecparallel_static)
ELSE(LINK_LIBRARY_TYPE MATCHES "static")
  SET(CVODE_LIB sundials_cvode_shared)
  SET(NVECP_LIB sundials_nvecparallel_shared)
  SET(FNVECP_LIB sundials_fnvecparallel_shared)
ENDIF(LINK_LIBRARY_TYPE MATCHES "static")

# Only static FCMIX libraries are available

SET(FCVODE_LIB sundials_fcvode_static)

# Set-up linker flags and link libraries

SET(SUNDIALS_LIBS ${FCVODE_LIB} ${CVODE_LIB} ${FNVECP_LIB} ${NVECP_LIB} ${EXTRA_LINK_LIBS})
IF(LAPACK_FOUND)
  LIST(APPEND SUNDIALS_LIBS ${LAPACK_LIBRARIES})
ENDIF(LAPACK_FOUND)

IF(KLU_FOUND)
  LIST(APPEND SUNDIALS_LIBS ${KLU_LIBRARIES})
ENDIF(KLU_FOUND)

IF(SUPERLUMT_FOUND)
  LIST(APPEND SUNDIALS_LIBS ${SUPERLUMT_LIBRARIES})
ENDIF(SUPERLUMT_FOUND)

IF(HYPRE_FOUND)
  LIST(APPEND SUNDIALS_LIBS ${HYPRE_LIBRARIES})
ENDIF(HYPRE_FOUND)

# Add the build and install targets for each CVODE example

FOREACH(example_tuple ${FCVODE_examples})
  list(GET example_tuple 0 example)
  list(GET example_tuple 1 number_of_nodes)
  list(GET example_tuple 2 number_of_tasks)
  # first item is example
  ADD_EXECUTABLE(${example} ${example}.f)
  SET_TARGET_PROPERTIES(${example} PROPERTIES FOLDER "Examples")
  SUNDIALS_ADD_TEST(${example} ${example} MPI_NPROCS ${number_of_tasks})
  TARGET_LINK_LIBRARIES(${example} ${SUNDIALS_LIBS})
  IF(NOT MPI_MPIF77)
    TARGET_LINK_LIBRARIES(${example} ${MPI_LIBRARY} ${MPI_EXTRA_LIBRARIES})
  ENDIF(NOT MPI_MPIF77)
  IF(EXAMPLES_INSTALL)
    INSTALL(FILES ${example}.f ${example}.out DESTINATION ${EXAMPLES_INSTALL_PATH}/cvode/fcmix_parallel)
  ENDIF(EXAMPLES_INSTALL)
ENDFOREACH(example_tuple ${FCVODE_examples})

IF(EXAMPLES_INSTALL)

  # Install the README file
  INSTALL(FILES README DESTINATION ${EXAMPLES_INSTALL_PATH}/cvode/fcmix_parallel)

  # Prepare substitution variables for Makefile and/or CMakeLists templates
  SET(SOLVER "CVODE")
  SET(SOLVER_LIB "sundials_cvode")
  SET(SOLVER_FLIB "sundials_fcvode")
  FOREACH(example_tuple ${FCVODE_examples})
    list(GET example_tuple 0 example)
    LIST2STRING(example EXAMPLES)
  ENDFOREACH(example_tuple ${FCVODE_examples})
  
  # Regardless of the platform we're on, we will generate and install 
  # CMakeLists.txt file for building the examples. This file  can then 
  # be used as a template for the user's own programs.

  # generate CMakelists.txt in the binary directory
  CONFIGURE_FILE(
      ${PROJECT_SOURCE_DIR}/examples/templates/cmakelists_parallel_F77_ex.in
      ${PROJECT_BINARY_DIR}/examples/cvode/fcmix_parallel/CMakeLists.txt
      @ONLY
      )

  # install CMakelists.txt
  INSTALL(
    FILES ${PROJECT_BINARY_DIR}/examples/cvode/fcmix_parallel/CMakeLists.txt
    DESTINATION ${EXAMPLES_INSTALL_PATH}/cvode/fcmix_parallel 
    )

  # On UNIX-type platforms, we also  generate and install a makefile for 
  # building the examples. This makefile can then be used as a template 
  # for the user's own programs.

  IF(UNIX)
    # generate Makefile and place it in the binary dir
    CONFIGURE_FILE(
      ${PROJECT_SOURCE_DIR}/examples/templates/makefile_parallel_F77_ex.in
      ${PROJECT_BINARY_DIR}/examples/cvode/fcmix_parallel/Makefile_ex
      @ONLY
      )
    # install the configured Makefile_ex as Makefile
    INSTALL(
      FILES ${PROJECT_BINARY_DIR}/examples/cvode/fcmix_parallel/Makefile_ex
      DESTINATION ${EXAMPLES_INSTALL_PATH}/cvode/fcmix_parallel 
      RENAME Makefile
      )
  ENDIF(UNIX)

ENDIF(EXAMPLES_INSTALL)
