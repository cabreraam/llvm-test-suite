################################################################################
#
# CMakeLists.txt
# 
# Author: Anthony Cabrera:
# Contact: cabreraam AT ieee DOT org
# Description: CMakeLists.txt replacement for SNAP Makefile
#
################################################################################

cmake_minimum_required(VERSION 3.18)


# Direct CMake to a specific MPI implementation. Specify
# -DMPI_EXECUTABLE_SUFFIX=.mpich for MPICH MPI implementation 
#set(MPI_EXECUTABLE_SUFFIX "" CACHE STRING "Direct CMake to a specific MPI implementation")

add_subdirectory(src)
