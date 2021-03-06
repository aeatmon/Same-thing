SET(pio_include_dirs_  ${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_CURRENT_BINARY_DIR})

OPTION(PIO_BIG_ENDIAN "Specify that the machine is big endian" "test")
IF ("${PIO_BIG_ENDIAN}" STREQUAL "test") 
  INCLUDE(TestBigEndian)
  TestBigEndian(PIO_BIG_ENDIAN_TEST)
  IF(PIO_BIG_ENDIAN_TEST) 
    SET(PIO_BIG_ENDIAN ON  CACHE BOOL "")
  ELSE()
    SET(PIO_BIG_ENDIAN OFF CACHE BOOL "")
  ENDIF()
ELSE()
  SET(PIO_BIG_ENDIAN ${PIO_BIG_ENDIAN} CACHE BOOL "")
ENDIF()                                            

SET(bld_PIO_DEFINITIONS)

IF (PIO_FILESYSTEM_HINTS STREQUAL "lustre")
  MESSAGE(STATUS "PIO using lustre filesystem hints")
  SET(bld_PIO_DEFINITIONS ${bld_PIO_DEFINITIONS} -DPIO_LUSTRE_HINTS)
ELSEIF(PIO_FILESYSTEM_HINTS STREQUAL "gpfs")
  MESSAGE(STATUS "PIO using gpfs filesystem hints")
  SET(bld_PIO_DEFINITIONS ${bld_PIO_DEFINITIONS} -DPIO_GPFS_HINTS)
ELSEIF(NOT "${PIO_FILESYSTEM_HINTS}" STREQUAL "")
  MESSAGE(WARNING "${PIO_FILESYSTEM_HINTS} not valid option for PIO_FILESYSTEM_HINTS; use gpfs or lustre.")
ENDIF()

IF(Netcdf_FOUND) 
  SET(pio_include_dirs_ ${pio_include_dirs_} ${Netcdf_INCLUDE_DIR})
  SET(bld_PIO_DEFINITIONS ${bld_PIO_DEFINITIONS} -D_NETCDF)
ELSE()
  SET(bld_PIO_DEFINITIONS ${bld_PIO_DEFINITIONS} -D_NONETCDF)
ENDIF()
IF(Pnetcdf_FOUND)
  SET(pio_include_dirs_ ${pio_include_dirs_} ${Pnetcdf_INCLUDE_DIR})
  SET(bld_PIO_DEFINITIONS ${bld_PIO_DEFINITIONS} -D_PNETCDF)
ELSE()
  SET(bld_PIO_DEFINITIONS ${bld_PIO_DEFINITIONS} -D_NOPNETCDF)
ENDIF()

OPTION(PIO_USE_MPIIO "Enable support for MPI-IO (default auto detect)" ON)
IF (PIO_USE_MPIIO) 
  TRY_COMPILE(MPIIO_SUCCESS ${CMAKE_CURRENT_BINARY_DIR}/tryCompileMPIIO 
                            ${CMAKE_CURRENT_SOURCE_DIR}/cmake/TryMPIIO.f90)
  IF (${MPIIO_SUCCESS})
    MESSAGE(STATUS "MPIIO detected and enabled.")
  ELSE()
    MESSAGE(STATUS "MPIIO not detected and therefore disabled.")
    SET(PIO_USE_MPIIO FALSE)
  ENDIF()
ENDIF()
IF (${PIO_USE_MPIIO}) 
  SET(bld_PIO_DEFINITIONS ${bld_PIO_DEFINITIONS} -DUSEMPIIO)
ENDIF()

SET(bld_PIO_DEFINITIONS ${bld_PIO_DEFINITIONS} -D_NOUSEMCT)

OPTION(PIO_USE_BOX  "" ON)
if(PIO_USE_BOX)
  SET(bld_PIO_DEFINITIONS ${bld_PIO_DEFINITIONS} -D_USEBOX)
else()
  SET(bld_PIO_DEFINITIONS ${bld_PIO_DEFINITIONS} -D_NOUSEBOX)
endif()

OPTION(PIO_USE_MPIMOD "Use Fortran MPI module (default auto detect)" ON)
IF (PIO_USE_MPIMOD)
  TRY_COMPILE(MPIMOD_SUCCESS ${CMAKE_CURRENT_BINARY_DIR}/tryCompileMPIMod
                             ${CMAKE_CURRENT_SOURCE_DIR}/cmake/TryMPIMod.f90)
  IF (${MPIMOD_SUCCESS})
    MESSAGE(STATUS "MPI Fortran module detected and enabled.")
  ELSE()
    MESSAGE(STATUS "MPI Fortran module not detected and therefore disabled.")
    SET(PIO_USE_MPIMOD FALSE)
  ENDIF()
ENDIF()

IF (NOT ${PIO_USE_MPIMOD})
  SET(bld_PIO_DEFINITIONS ${bld_PIO_DEFINITIONS} -DNO_MPIMOD)
ENDIF()

SET(PIO_DEFINITIONS ${bld_PIO_DEFINITIONS} CACHE STRING "")
ADD_DEFINITIONS(${PIO_DEFINITIONS})

SET(PIO_INCLUDE_DIRS   ${pio_include_dirs_} CACHE STRING "")
INCLUDE_DIRECTORIES(${PIO_INCLUDE_DIRS})

SET(SRCS_C topology.c)

SET(SRCS_F90 pio.F90 pio_kinds.F90 nf_mod.F90  ionf_mod.F90 pio_types.F90
             piolib_mod.F90 pio_mpi_utils.F90 pio_nf_utils.F90 pio_utils.F90
             pio_quicksort.F90 pio_msg_mod.F90 pio_msg_callbacks.F90)

SET(TEMPSRCF90    pionfatt_mod.F90
         	                pionfread_mod.F90
                                pionfwrite_mod.F90
                                pionfput_mod.F90
                                pionfget_mod.F90
	                        alloc_mod.F90
                                box_rearrange.F90
                                rearrange.F90
                                pio_support.F90
         	                iompi_mod.F90
           	                piodarray.F90
	                        pio_spmd_utils.F90
	                        pio_msg_getput_callbacks.F90
                                calcdisplace_mod.F90
                                calcdecomp.F90
        )

ADD_LIBRARY(pio ${SRCS_F90} ${SRC_C} ${TEMPSRCF90})
TARGET_LINK_LIBRARIES(pio ${Pnetcdf_LIBRARIES})
TARGET_LINK_LIBRARIES(pio ${Netcdf_LIBRARIES})
