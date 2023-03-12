cmake_minimum_required(VERSION 3.1)
project(traypp VERSION 0.2 DESCRIPTION "A cross-platform C++ system tray library")

file(GLOB_RECURSE tray_src "tray/src/*.cpp")
add_library(traypp STATIC ${tray_src})
add_library(traypp::traypp ALIAS traypp)

if (UNIX)
    find_package(PkgConfig REQUIRED)
    pkg_check_modules(GTK3 REQUIRED gtk+-3.0)
    pkg_check_modules(APPINDICATOR REQUIRED appindicator3-0.1)

    target_link_libraries(traypp INTERFACE ${GTK3_LIBRARIES} ${APPINDICATOR_LIBRARIES})

    if (CMAKE_BUILD_TYPE STREQUAL "Debug")
        target_compile_options(traypp PRIVATE -Wall -Wextra -Werror -pedantic -Wno-unused-lambda-capture)
    endif()

    target_include_directories(traypp SYSTEM PUBLIC ${GTK3_INCLUDE_DIRS} ${APPINDICATOR_INCLUDE_DIRS} ${PROJECT_SOURCE_DIR})
endif()

target_include_directories(traypp SYSTEM PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/tray/include> $<INSTALL_INTERFACE:include>)

target_compile_features(traypp PRIVATE cxx_std_17)
set_target_properties(traypp PROPERTIES
                      CXX_STANDARD 17
                      CXX_EXTENSIONS OFF
                      CXX_STANDARD_REQUIRED ON)

set_target_properties(traypp PROPERTIES VERSION ${PROJECT_VERSION})
set_target_properties(traypp PROPERTIES PROJECT_NAME ${PROJECT_NAME})

install(TARGETS traypp
EXPORT TrayppTargets
    RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}"
    ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}"
    LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}"
)
install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/tray/include/ DESTINATION include/traypp)

install(EXPORT TrayppTargets
    FILE TrayppTargets.cmake
    NAMESPACE traypp::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/traypp
)

include(CMakePackageConfigHelpers)
configure_package_config_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/TrayppConfig.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/TrayppConfig.cmake"
    INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/traypp
)
install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/TrayppConfig.cmake"
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/traypp
)
