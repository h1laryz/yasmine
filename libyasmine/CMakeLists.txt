cmake_minimum_required(VERSION 2.8)

project(libyasmine CXX)

find_package(libessentials CONFIG REQUIRED)
find_package(libhermes CONFIG REQUIRED)

set(CMAKE_VERBOSE_MAKEFILE ON)

if(POLICY CMP0054)
	cmake_policy(SET CMP0054 NEW)
endif()

file(GLOB_RECURSE yasmine_SRC "source/*.cpp")

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

include(GNUInstallDirs)
include(./../CMakeHelpers/addSpecificFlags.cmake)
include(./../CMakeHelpers/addYasmineOptions.cmake)
include(./../CMakeHelpers/functions.cmake)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${extra_flags}")

if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU" OR "${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
	add_library(libyasmine ${LINK_TYPE} ${yasmine_SRC})
elseif("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
	add_library(libyasmine STATIC ${yasmine_SRC})
endif()

target_include_directories(libyasmine PUBLIC
		$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
		$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include_impl>
		$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/yasmine/>
)

set(link_libraries_list "")

function(use_pthread varName)
	if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU" OR "${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
		set(${varName} ${${varName}} pthread PARENT_SCOPE)
	endif()
endfunction()

if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU" OR "${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
	add_library_for_linking(link_libraries_list libessentials::libessentials)
	add_library_for_linking(link_libraries_list libhermes::libhermes)
else()
	add_library_for_linking(link_libraries_list libessentials::libessentials)
	add_library_for_linking(link_libraries_list libhermes::libhermes)
endif()

message(STATUS "libyasmine linking to libraries: ${link_libraries_list}")
target_link_libraries(libyasmine PUBLIC ${link_libraries_list})

set_target_properties(libyasmine PROPERTIES
		VERSION 1.3.1
		SOVERSION 1.3
		PREFIX ""
)

file(GLOB yasmine_hpp "include/*.hpp")
file(GLOB yasmine_hpp_impl "include_impl/*.hpp")

install(TARGETS libyasmine
		EXPORT libyasmine-targets
		ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
		LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
		RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)

install(FILES ${yasmine_hpp} ${yasmine_hpp_impl} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/yasmine/)

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
		"${CMAKE_CURRENT_BINARY_DIR}/libyasmine/libyasmineConfigVersion.cmake"
		VERSION 1.3.1
		COMPATIBILITY AnyNewerVersion
)

export(EXPORT libyasmine-targets
		FILE "${CMAKE_CURRENT_BINARY_DIR}/libyasmine/libyasmineTargets.cmake"
		NAMESPACE libyasmine::
)

configure_package_config_file(
		"${CMAKE_CURRENT_SOURCE_DIR}/Config.cmake.in"
		"${CMAKE_CURRENT_BINARY_DIR}/libyasmine/libyasmineConfig.cmake"
		INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/libyasmine
)

install(EXPORT libyasmine-targets
		FILE libyasmineTargets.cmake
		NAMESPACE libyasmine::
		DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/libyasmine
)

install(
		FILES
		"${CMAKE_CURRENT_BINARY_DIR}/libyasmine/libyasmineConfig.cmake"
		"${CMAKE_CURRENT_BINARY_DIR}/libyasmine/libyasmineConfigVersion.cmake"
		DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/libyasmine
)