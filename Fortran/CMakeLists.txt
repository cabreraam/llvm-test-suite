# Fortran TestSuite Directory Structure (TENTATIVE):
#
# UnitTests    - Single source tests for language features
# Benchmarks   - Single source tests for benchmarking
# Applications - Multi-source tests for proxy apps, small apps, etc.
#
# This file should only contain add_subdirectory(...) one for each test
# directory
include(ExternalProject)

option(PGMATH OFF "Whether to use libpgmath or not. Useful for Flang") 

if (PGMATH)
	set(PGMATH_PREFIX "${CMAKE_BINARY_DIR}/Fortran/pgmath")
	ExternalProject_Add(
		pgmath
	
		SOURCE_DIR "${CMAKE_SOURCE_DIR}/Fortran/pgmath"
		SOURCE_SUBDIR "runtime/libpgmath"
	
		GIT_REPOSITORY "https://github.com/flang-compiler/flang.git"
		GIT_TAG "master"
		GIT_SHALLOW TRUE
		GIT_PROGRESS FALSE
	
		PREFIX pgmath
		CMAKE_ARGS
			-DCMAKE_C_COMPILER=${CMAKE_C_COMPILER}
			-DCMAKE_CXX_COMPILER=${CMAKE_CXX_COMPILER}
			-DCMAKE_INSTALL_PREFIX=${PGMATH_PREFIX}
	
		BUILD_BYPRODUCTS ${PGMATH_PREFIX}/lib/libpgmath.so
	)

	set(PGMATH_DIR "${PGMATH_PREFIX}/lib"
		CACHE
		STRING
		"Path to installation directory for PGMATH"
	)
	
	add_custom_command(
		TARGET
			pgmath
		POST_BUILD
		COMMAND
			cmake -E remove_directory ${CMAKE_BINARY_DIR}/thirdparty/pgmath/test
		COMMAND
			cmake -E remove_directory ${BINARY_DIR}/test
		COMMAND
			cmake -E remove -f ${CMAKE_BINARY_DIR}/thirdparty/pgmath/test
		COMMAND
			cmake -E remove -f ${BINARY_DIR}/test
		VERBATIM
	)
endif()

add_subdirectory(UnitTests)
add_subdirectory(SNAP)
