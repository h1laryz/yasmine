cmake_minimum_required(VERSION 3.0)  # Updated to a more modern version

project(libhermes)

# Ensure libessentials is found. Ensure this package exports its targets correctly.
find_package(libessentials REQUIRED)

set(CMAKE_VERBOSE_MAKEFILE ON)

# Set global C++ standard rather than rely on if conditions on CXX_FLAGS directly.
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

file(GLOB_RECURSE hermes_SRC "source/*.cpp")

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

include(GNUInstallDirs)

# Assuming these modules set necessary compiler flags.
include(./../CMakeHelpers/addSpecificFlags.cmake)
include(./../CMakeHelpers/addSeadexOptions.cmake)

add_library(${PROJECT_NAME} ${hermes_SRC})
target_link_libraries(${PROJECT_NAME} PUBLIC libessentials::libessentials)
target_include_directories(${PROJECT_NAME} PUBLIC
		$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
		$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>)

# Gather public header files and install them.
file(GLOB hermes_hpp "include/hermes/*.hpp")
install(FILES ${hermes_hpp} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/hermes)

install(TARGETS ${PROJECT_NAME}
		EXPORT ${PROJECT_NAME}-targets
		ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
		LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
		RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

install(EXPORT ${PROJECT_NAME}-targets
		FILE ${PROJECT_NAME}Targets.cmake
		NAMESPACE ${PROJECT_NAME}::
		DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME})

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
		${PROJECT_NAME}ConfigVersion.cmake
		VERSION 1.0
		COMPATIBILITY AnyNewerVersion)

configure_package_config_file(Config.cmake.in
		${PROJECT_NAME}Config.cmake
		INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME})

install(FILES
		"${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
		"${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
		DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME})

set_target_properties(${PROJECT_NAME} PROPERTIES PREFIX "")