cmake_minimum_required(VERSION 3.0)  # Updated for consistency and modern features

project(libessentials)

include(GNUInstallDirs)

set(CMAKE_VERBOSE_MAKEFILE ON)

file(GLOB_RECURSE essentials_SRC "source/*.cpp")

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

include(./../../../../CMakeHelpers/addSpecificFlags.cmake)
include(./../../../../CMakeHelpers/addSeadexOptions.cmake)

add_library(${PROJECT_NAME} ${essentials_SRC})

target_include_directories(${PROJECT_NAME} PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>)

file(GLOB essentials_hpp "include/essentials/*.hpp")
file(GLOB essentials_comp_hpp "include/essentials/compatibility/*.hpp")

# Ensure the compatibility headers are installed in the correct subdirectory.
install(FILES ${essentials_hpp} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/essentials)
install(FILES ${essentials_comp_hpp} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/essentials/compatibility)

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