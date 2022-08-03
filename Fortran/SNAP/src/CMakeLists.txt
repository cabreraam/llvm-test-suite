################################################################################
#
# CMakeLists.txt
# 
# Author: Anthony Cabrera:
# Contact: cabreraam AT ieee DOT org
# Description: CMakeLists.txt replacement for SNAP Makefile
#
################################################################################

################################################################################
#
# Options and Variables
#
################################################################################


option(SNAP_MPI "Use MPI for the fortran SNAP test" OFF)
option(SNAP_OPT "Turn on optimizations for the fortran SNAP test" ON)
option(SNAP_OPENMP "Enable OpenMP for the fortran SNAP test" OFF)
option(SNAP_KNL "Intel KNL target for the fortran SNAP test" OFF)
option(SNAP_HASWELL 
	"Does the CPU use the Haswell u-arch for the fortran SNAP test" 
	OFF)
option(SNAP_CRAY 
	"Target system is a Cray machine for the fortran SNAP test"
	OFF)

if(SNAP_MPI)
	find_package(MPI REQUIRED)
	message(STATUS "MPI_Fortran_COMPILER - ${MPI_Fortran_COMPILER}")
	message(STATUS "MPI_Fortran_COMPILE_OPTIONS - ${MPI_Fortran_COMPILE_OPTIONS}")
	message(STATUS "MPI_Fortran_COMPILE_DEFINITIONS - ${MPI_Fortran_COMPILE_DEFINITIONS}")
	message(STATUS "MPI_Fortran_INCLUDE_DIRS - ${MPI_Fortran_INCLUDE_DIRS}")
	message(STATUS "MPI_Fortran_LINK_FLAGS - ${MPI_Fortran_LINK_FLAGS}")
	message(STATUS "MPI_Fortran_LIB_NAMES - ${MPI_Fortran_LIB_NAMES}")
	message(STATUS "MPI_Fortran_LIBRARY - ${MPI_Fortran_LIBRARY}")
	message(STATUS "MPI_Fortran_LIBRARIES - ${MPI_Fortran_LIBRARIES}")
endif()

if(SNAP_OPENMP)
	find_package(OpenMP REQUIRED)
	message(STATUS "OpenMP_VERSION - ${OpenMP_VERSION}")
	message(STATUS "OpenMP_Fortran_FOUND - ${OpenMP_Fortran_COMPILE_OPTIONS}")
	message(STATUS "OpenMP_Fortran_FLAGS - ${OpenMP_Fortran_COMPILE_DEFINITIONS}")
	message(STATUS "OpenMP_Fortran_INCLUDE_DIRS - ${OpenMP_Fortran_INCLUDE_DIRS}")
	message(STATUS "OpenMP_Fortran_LINK_FLAGS - ${OpenMP_Fortran_LINK_FLAGS}")
	message(STATUS "OpenMP_Fortran_LIB_NAMES - ${OpenMP_Fortran_LIB_NAMES}")
	message(STATUS "OpenMP_Fortran_LIBRARY - ${OpenMP_Fortran_LIBRARY}")
	message(STATUS "OpenMP_Fortran_LIBRARIES - ${OpenMP_Fortran_LIBRARIES}")
	message(STATUS "OpenMP_HOME - ${OpenMP_HOME}")
endif()

# Compiler Preprocessor Definitions
# These should be passed in at the command line as a semi-colon-separated list
# If the developer wants MPI, it would be set here
# e.g., MPI + other pound define --> -DUSER_DEFS="MPI;OTHER_VAR;OTHER_VAR2=1"
set(USER_DEFS "" CACHE STRING "Semi-colon separted preprocessor pound defines")

set(PGM_DIR "" CACHE STRING "Location of libpgmath if using LLVM flang")

set(FP_TOLERANCE "0.001" CACHE STRING "FP tolerance value")


################################################################################
#
# Compile Options for Different Compilers
#
################################################################################

# Compile Options for GNU
set(SNAP_COMPILER_OPTIONS_GNU_OPT
	"-O3" 
	CACHE 
	STRING 
	"GNU compiler options when SNAP_OPT=ON"
)
set(SNAP_COMPILER_OPTIONS_GNU_NOT_OPT
	"-O0"
	"-g"
	"-fbounds-check"
	"-fbacktrace"
	CACHE 
	STRING 
	"GNU compiler options when SNAP_OPT=OFF"
)
set(SNAP_COMPILER_OPTIONS_GNU
	$<$<BOOL:${SNAP_OPENMP}>:-fopenmp>
	$<IF:$<BOOL:${SNAP_OPT}>,${SNAP_COMPILER_OPTIONS_GNU_OPT},${SNAP_COMPILER_OPTIONS_GNU_NOT_OPT}>
	CACHE
	STRING
	"Compiler options when GNU compiler detected"
)

# Compile Options for ifort 
set(SNAP_COMPILER_OPTIONS_IFORT_OPT
	"-O3"
	"-ip"
	"-align array32byte"
	"-qno-opt-dynamic-align"
	"-fno-fnalias"
	"-fp-model fast"
	"-fp-speculation fast"
	CACHE 
	STRING 
	"ifort compiler options when SNAP_OPT=ON"
)
set(SNAP_COMPILER_OPTIONS_INTEL_NOT_OPT
	"-O0"
	"-g "
	"-check bounds"
	"-traceback"
	"-warn unused"
	CACHE 
	STRING 
	"ifort compiler options when SNAP_OPT=OFF"
)
set(SNAP_COMPILER_OPTIONS_IFORT
	$<$<BOOL:${SNAP_OPENMP}>:-qopenmp>
	$<IF:$<BOOL:${SNAP_OPT}>,${SNAP_COMPILER_OPTIONS_IFORT_OPT},${SNAP_COMPILER_OPTIONS_INTEL_NOT_OPT}>
	$<IF:$<BOOL:${SNAP_HASWELL}>,"-xcore-avx2","-xHost">
	CACHE
	STRING
	"Compiler options when ifort compiler detected"
)

# Compile Options for ifort knl
set(SNAP_COMPILER_OPTIONS_IFORT_KNL_OPT
	"-O3"
	"-xmic-avx512"
	"-ip"
	"-align array64byte"
	"-qno-opt-dynamic-align"
	"-fp-model fast"
	"-fp-speculation fast"
	"-fno-alias -fno-fnalias"
	CACHE 
	STRING 
	"ifort compiler options when SNAP_OPT=ON and KNL=ON"
)
# This is the same as SNAP_COMPILER_OPTIONS_INTEL_NOT_OPT
#set(SNAP_COMPILER_OPTIONS_IFORT_KNL_NOT_OPT
#)
set(SNAP_COMPILER_OPTIONS_IFORT_KNL
	$<$<BOOL:${SNAP_OPENMP}>:-qopenmp>
	$<IF:$<BOOL:${SNAP_OPT}>,${SNAP_COMPILER_OPTIONS_IFORT_KNL_OPT},${SNAP_COMPILER_OPTIONS_INTEL_NOT_OPT}>
	CACHE
	STRING
	"Compiler options when ifort + knl compiler detected"
)

# Compile Options for intel ifx
set(SNAP_COMPILER_OPTIONS_IFX_OPT
	"-xcore-avx2"
	"-Ofast"
	"-funroll-loops"
	"-nostandard-realloc-lhs"
	"-align array32byte"
	CACHE 
	STRING 
	"ifx compiler options when SNAP_OPT=ON"
)
# This is the same as SNAP_COMPILER_OPTIONS_INTEL_NOT_OPT
#set(SNAP_COMPILER_OPTIONS_IFX_OPT
#)
set(SNAP_COMPILER_OPTIONS_IFX
	$<$<BOOL:${SNAP_OPENMP}>:-fiopenmp>
	$<IF:$<BOOL:${SNAP_OPT}>,${SNAP_COMPILER_OPTIONS_IFX_OPT},${SNAP_COMPILER_OPTIONS_INTEL_NOT_OPT}>
	CACHE
	STRING
	"Compiler options when ifx compiler detected"
)

# Compile Options for LLVM FLang
set(SNAP_COMPILER_OPTIONS_LLVM_FLANG_OPT
	"" 
	CACHE 
	STRING 
	"LLVMFlang compiler options when SNAP_OPT=ON"
)
set(SNAP_COMPILER_OPTIONS_LLVM_FLANG_NOT_OPT
	""
	CACHE 
	STRING 
	"LLVMFlang compiler options when SNAP_OPT=OFF"
)
set(SNAP_COMPILER_OPTIONS_LLVM_FLANG
	$<$<BOOL:${SNAP_OPENMP}>:""> # what is the OpenMP flag for LLVMFlang?
	$<IF:$<BOOL:${SNAP_OPT}>,${SNAP_COMPILER_OPTIONS_LLVM_FLANG_OPT},${SNAP_COMPILER_OPTIONS_LLVM_FLANG_NOT_OPT}>
	CACHE
	STRING
	"Compiler options when GNU compiler detected"
)

################################################################################
#
# Setting up build target
#
################################################################################

set(SNAP_SOURCE_FILES
	global.f90
	snap_main.f90
	utils.f90
	version.f90
	plib.F90
	geom.f90
	sn.f90
	data.f90
	control.f90
	input.f90
	setup.f90
	dealloc.f90
	translv.f90
	solvar.f90
	outer.f90
	expxs.f90
	inner.f90
	sweep.f90
	octsweep.f90
	dim1_sweep.f90
	dim3_sweep.f90
	output.f90
	time.F90
	mms.f90
	analyze.f90
	thrd_comm.f90
	mkba_sweep.f90
	CACHE
	STRING
	"List of SNAP source files"
)

add_executable(snap
	${SNAP_SOURCE_FILES}
)

if (PGMATH)
	add_dependencies(snap pgmath)
endif()

target_compile_options(snap 
	PRIVATE
		$<$<Fortran_COMPILER_ID:GNU>:${SNAP_COMPILER_OPTIONS_GNU}>
		$<$<AND:$<Fortran_COMPILER_ID:Intel>,$<NOT:$<BOOL:${SNAP_KNL}>>>:${SNAP_COMPILER_OPTIONS_IFORT}>
		$<$<AND:$<Fortran_COMPILER_ID:Intel>,$<BOOL:${SNAP_KNL}>>:${SNAP_COMPILER_OPTIONS_IFORT_KNL}>
		$<$<Fortran_COMPILER_ID:IntelLLVM>:${SNAP_COMPILER_OPTIONS_IFX}>
		$<$<Fortran_COMPILER_ID:LLVMFlang>:${SNAP_COMPILER_OPTIONS_LLVM_FLANG}>
)
target_compile_definitions(snap
	PRIVATE
		${USER_DEFS}
		$<$<AND:$<BOOL:${SNAP_MPI}>,$<BOOL:${SNAP_CRAY}>>:MPI>
		$<$<BOOL:${SNAP_OPENMP}>:OPENMP>
)
# -I$(MPICH_DIR)/include
target_include_directories(snap
	PRIVATE
		$<$<AND:$<BOOL:${SNAP_MPI}>,$<BOOL:${SNAP_CRAY}>>:${MPI_Fortran_INCLUDE_DIRS}>
		$<$<BOOL:${SNAP_OPENMP}>:${OpenMP_Fortran_INCLUDE_DIRS}>
)
target_link_libraries(snap
	PRIVATE
		"$<$<AND:$<BOOL:${SNAP_MPI}>,$<BOOL:${SNAP_CRAY}>>:${MPI_Fortran_LIBRARIES}>"
		"$<$<BOOL:${SNAP_OPENMP}>:${OpenMP_Fortran_LIBRARIES}>"
		"$<$<BOOL:${PGMATH}>:${PGMATH_DIR}/libpgmath.so>"
)

macro(llvm_test_verify_hash_program_output _file)
  llvm_test_verify(%b/HashProgramOutput.sh ${_file})
endmacro()

# This if-body takes care of all of the llvm-test suite stuff. It is modeled
# after the llvm_multisource function in the llvm-test-suite, which takes the
# following form:
#function(llvm_multisource target)
#  set(sources ${ARGN})
#  if(NOT sources)
#    file(GLOB sources *.c *.cpp *.cc *.f *.F *.f90 *.F90)
#  endif()
#  llvm_test_executable_no_test(${target} ${sources})
#  llvm_test_traditional(${target})
#  llvm_add_test_for_target(${target})
#endfunction()
# Because 
# - I could not directly use those pre-existing functions/macros, 
# - I wanted to reference the work I'd already done with the SNAP CMake infra
# I will show how my implementation below corresponds to llvm_multisource. Look
# for comments that take the form  # start <function>
if(PROJECT_NAME STREQUAL "test-suite")
	set(target "snap")

  # start llvm_test_executable_no_test_fortran
	# Note: we ignore TEST_SUITE_PROFILE_USE. Will this break LLVM Flang? if we
	# aded that flag?
  llvm_codesign(snap)
  set_property(GLOBAL APPEND PROPERTY TEST_SUITE_TARGETS snap)
  test_suite_add_build_dependencies(snap)

  if(TEST_SUITE_LLVM_SIZE)
    add_custom_command(TARGET ${target} POST_BUILD
      COMMAND ${TEST_SUITE_LLVM_SIZE} --format=sysv $<TARGET_FILE:${target}>
      > $<TARGET_FILE:${target}>.size)
  endif()

  # start llvm_test_traditional(${target})
	# isolate path for the input file that we use	
	set(TEST_INPUT_DIR 
		"${CMAKE_CURRENT_SOURCE_DIR}/../qasnap/mms_src"
	)
	message(STATUS "TEST_INPUT_DIR - ${TEST_INPUT_DIR}")
	set(TEST_INPUT_FILENAME "2d_mms_st.inp")
	set(TEST_OUTPUT_FILENAME "snap-output")
	llvm_test_data(${target} 
		MUST_COPY
		SOURCE_DIR
			${TEST_INPUT_DIR}
		${TEST_INPUT_FILENAME}
	)
	# make run command include the new input we just copied
	list(APPEND RUN_OPTIONS 
		# the following line is commented out because in global.f90, the ifile
		# char buffer is 64 bytes long! without the comment, the snap binary cannot
		# read the input file! Since we change directory to where the binary lives,
		# and we copy the input file directly to the binary directory, just
		# specifying the input filename works perfrectly fine.
		#%S/${TEST_INPUT_FILENAME} 
		${TEST_INPUT_FILENAME} 
		${TEST_OUTPUT_FILENAME}
	)
  list(INSERT RUN_OPTIONS 0 WORKDIR %S)
	# here, we take the options for the RUN: command that we want to craft, and
	# pass it to the llvm_test_run fxn
  llvm_test_run(${RUN_OPTIONS})

  # Hash if we've been asked to.
  if(HASH_PROGRAM_OUTPUT)
    llvm_test_verify_hash_program_output(%o)
  endif()
	
	# Deal with output that we want to compare to
	if(EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/snap.reference_output)
		set(REFERENCE_OUTPUT 
			snap.reference_output
		)
		set(DIFFPROG_BASE %b/${FPCMP})
		set(FP_TOL_SUFFIX "-r ${FP_TOLERANCE}")
		set(FP_ABSTOL_SUFFIX "-a ${FP_ABSTOLERANCE}")
		set(DIFFPROG
			${DIFFPROG_BASE}
			"$<$<BOOL:${FP_TOLERANCE}>:${FP_TOL_SUFFIX}>"
			"$<$<BOOL:${FP_ABSTOLERANCE}>:${FP_ABSTOL_SUFFIX}>"
		)

		llvm_test_verify(${DIFFPROG} %o %S/${REFERENCE_OUTPUT})
		llvm_test_data(${target} ${REFERENCE_OUTPUT})
	else()
		message("-- No reference output found for test ${name}")
	endif()
  set(TESTSCRIPT "${TESTSCRIPT}" PARENT_SCOPE)

	# this creates the .test file for you!
	# start llvm_add_test_for_target(${target})
	# don't need to make any mods to this, woo!
  llvm_add_test_for_target(snap)

	# lit.local.cfg has since been added to the src directory. Additionally,
	# we'll need to copy that file over to the binary directory. 
	file(COPY lit.local.cfg DESTINATION "${CMAKE_CURRENT_BINARY_DIR}")
endif()

