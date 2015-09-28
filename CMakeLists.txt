# This is a minimalist cmake file designed to be used
# with libigl. If you need CoMISo for other purposes
# please use the CMakeLists_original.cmake which
# supports all the features of CoMISo

cmake_minimum_required (VERSION 2.6)

project(CoMISo)

add_definitions(
    -DINCLUDE_TEMPLATES
)

if (WIN32)
  add_definitions(
      -D_USE_MATH_DEFINES -DNOMINMAX
  )
endif(WIN32)

add_library(CoMISo STATIC
Solver/EigenLDLTSolver.hh Solver/EigenLDLTSolver.cc
Solver/MISolver.hh Solver/MISolver.cc
Solver/ConstrainedSolver.hh Solver/ConstrainedSolver.cc
Solver/IterativeSolverT.hh Solver/IterativeSolverT.cc
Solver/GMM_Tools.hh Solver/GMM_Tools.cc
)
