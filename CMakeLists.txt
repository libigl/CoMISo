# This is a minimalist cmake file designed to be used
# with libigl. If you need CoMISo for other purposes
# please use the CMakeLists_original.cmake which
# supports all the features of CoMISo

cmake_minimum_required(VERSION 3.11)

project(CoMISo)

add_library(CoMISo STATIC
  Solver/EigenLDLTSolver.hh Solver/EigenLDLTSolver.cc
  Solver/MISolver.hh Solver/MISolver.cc
  Solver/ConstrainedSolver.hh Solver/ConstrainedSolver.cc
  Solver/IterativeSolverT.hh Solver/IterativeSolverT.cc
  Solver/GMM_Tools.hh Solver/GMM_Tools.cc)

target_include_directories(CoMISo PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/..>
  $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>)
target_include_directories(CoMISo PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/../eigen>
  $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/eigen>)
target_include_directories(CoMISo PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/ext/gmm-4.2/include>
  $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/ext/gmm-4.2/include>)

target_compile_definitions(CoMISo PUBLIC -DINCLUDE_TEMPLATES)

target_link_libraries(CoMISo PUBLIC Eigen3::Eigen)

# Generate position independent code
set_target_properties(CoMISo PROPERTIES POSITION_INDEPENDENT_CODE ON)

if(WIN32)
  target_compile_definitions(CoMISo PRIVATE -D_USE_MATH_DEFINES -DNOMINMAX)
  target_compile_options(CoMISo PUBLIC /D_SCL_SECURE_NO_DEPRECATE)
  # Deliberately do NOT link the OpenBLAS copy vendored under
  # ext/OpenBLAS-v0.2.14-Win64-int64: it is an INTERFACE64 (-i8) build whose Fortran
  # INTEGER is 64-bit, while gmm's BLAS/LAPACK interface (enabled by GMM_USES_LAPACK in
  # Solver/GMM_Tools.cc) declares and calls every routine with 32-bit ints. Handing the
  # address of a 32-bit int to a routine that reads 64 bits is undefined behaviour: the
  # routine reads the neighbouring stack, so lengths and increments become garbage and
  # BLAS runs out of bounds. It surfaces as a SIGSEGV that depends on the optimization
  # level -- the MIQ pipeline hits it on its first gmm::vect_sp. COMISO_NO_BLAS compiles
  # the interface out instead, letting gmm use its own implementations; it is PUBLIC
  # because GMM_Tools.cc is textually included by consumers (INCLUDE_TEMPLATES), so they
  # have to see the same setting, and because without it those consumers would reference
  # BLAS symbols nothing provides. Re-enable BLAS by pointing at a 32-bit-integer build
  # (and then shipping its runtime DLLs alongside the executables).
  target_compile_definitions(CoMISo PUBLIC -DCOMISO_NO_BLAS)
endif()

if(APPLE)
  find_library(accelerate_library Accelerate)
  target_link_libraries(CoMISo PUBLIC ${accelerate_library})
elseif(UNIX)
  find_package(BLAS REQUIRED)
  target_link_libraries(CoMISo PUBLIC ${BLAS_LIBRARIES})
endif()
