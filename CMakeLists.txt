cmake_minimum_required(VERSION 3.8)

project(app-streamer-Procedural)

set(CMAKE_CXX_STANDARD 17)

option(BUILD_STATIC "Build static" ON)
option(BUILD_SHARED "Build shared" ON)

############################### install hub

include(ExternalProject)
set(EXTERNAL_NAME hub)
find_package(${EXTERNAL_NAME} QUIET)
if(NOT ${EXTERNAL_NAME}_FOUND)
    list(
        APPEND
        CMAKE_ARGS
        "-DCMAKE_INSTALL_PREFIX:PATH=${CMAKE_CURRENT_BINARY_DIR}/external/src/${EXTERNAL_NAME}-install"
        "-DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}"
        "-DHUB_BUILD_STATIC=${BUILD_STATIC}"
        "-DHUB_BUILD_SHARED=${BUILD_SHARED}"
        )

    message(STATUS "Preparing external project \"${EXTERNAL_NAME}\" with args:")
    foreach(CMAKE_ARG ${CMAKE_ARGS})
        message(STATUS " ${CMAKE_ARG}")
    endforeach()

    ExternalProject_Add(
        ${EXTERNAL_NAME}
        GIT_REPOSITORY https://github.com/T-PLAY/hub.git
#        GIT_TAG main
        GIT_TAG installMinorFix
        GIT_SHALLOW ON
        GIT_PROGRESS FALSE
        PREFIX "external"
        CMAKE_ARGS "${CMAKE_ARGS}")

    set(${EXTERNAL_NAME}_DIR
        "${CMAKE_CURRENT_BINARY_DIR}/external/src/${EXTERNAL_NAME}-install/share/hub/cmake/"
        CACHE PATH "" FORCE)
endif()

#find_package(hub REQUIRED)
if (hub_FOUND)

    file(GLOB sources main.cpp)

    add_executable(${PROJECT_NAME} ${sources})
    target_link_libraries(${PROJECT_NAME} hub)

#    add_executable(${PROJECT_NAME}_static ${sources})
#    target_link_libraries(${PROJECT_NAME} hub_static)

#    add_executable(${PROJECT_NAME}_shared ${sources})
#    target_link_libraries(${PROJECT_NAME} hub_shared)

#    target_include_directories(${PROJECT_NAME} PUBLIC ${PROJECT_SOURCE_DIR})

    if(WIN32)
        add_custom_command(
            TARGET ${PROJECT_NAME}
            POST_BUILD
            COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:hub> $<TARGET_FILE_DIR:${PROJECT_NAME}>)
    endif()

endif()
