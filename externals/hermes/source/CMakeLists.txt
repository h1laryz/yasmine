cmake_minimum_required(VERSION 2.0)

if(POLICY CMP0054)
  cmake_policy(SET CMP0054 NEW)
endif()

include(./CMakeHelpers/setVariables.cmake)

include(./CMakeHelpers/addBoost.cmake)
include(./CMakeHelpers/functions.cmake)
include(./CMakeHelpers/functionsAddCompilerFlags.cmake)

add_subdirectory(hermes)
