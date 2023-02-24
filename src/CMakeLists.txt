#  Input variables:
#   EXTERNALS_INSTALL_DIR
#   CMAKE_PREFIX_PATH
#   CW_LIB_DIR

option (REBUILD_EXTERNAL_LIBS "Rebuild external libraries" OFF)

if("$ENV{REBUILD_EXTERNAL_LIBS}" STREQUAL "true")
    set(REBUILD_EXTERNAL_LIBS ON)
endif()

if(MSVC)
    set(EXTERNALS_BUILD_DIR $ENV{TEMP}/project_Hellgate)
    message("EXTERNAL BUILD DIR IS: $ENV{TEMP}/project_Hellgate")
else()
    set(EXTERNALS_BUILD_DIR ${CMAKE_BINARY_DIR})
endif()

if(MSVC AND NOT (${CMAKE_GENERATOR} MATCHES "Win64"))
    set(CMAKE_GENERATOR_PLATFORM -A ${CMAKE_GENERATOR_PLATFORM})
else()
    set(CMAKE_GENERATOR_PLATFORM)
endif()

#fmt

set(FMT_GIT_TAG "8.0.1")
set(FMT_INSTALL_DIR ${EXTERNALS_INSTALL_DIR}/fmt-${FMT_GIT_TAG})

if(NOT EXISTS "${FMT_INSTALL_DIR}/include/fmt/core.h")
    set(FMT_BUILD_DIRECTORY ${EXTERNALS_BUILD_DIR}/external/fmt)
    file(REMOVE_RECURSE ${FMT_BUILD_DIRECTORY})
    file(MAKE_DIRECTORY ${FMT_BUILD_DIRECTORY})
    execute_process(
        COMMAND ${CMAKE_COMMAND}
            -DFMT_GIT_TAG=${FMT_GIT_TAG}
            -DFMT_BUILD_DIR=${FMT_BUILD_DIRECTORY}
            -DFMT_INSTALL_DIR=${FMT_INSTALL_DIR}
            -G ${CMAKE_GENERATOR}
            ${CMAKE_GENERATOR_PLATFORM}
            -S ${CMAKE_SOURCE_DIR}/external/fmt
            -B ${FMT_BUILD_DIRECTORY}
        WORKING_DIRECTORY ${FMT_BUILD_DIRECTORY}
    )
    execute_process(
        COMMAND ${CMAKE_COMMAND}
            --build .
        WORKING_DIRECTORY ${FMT_BUILD_DIRECTORY}
    )
endif()

set(CMAKE_PREFIX_PATH_OLD ${CMAKE_PREFIX_PATH})
set(CMAKE_PREFIX_PATH ${FMT_INSTALL_DIR})
find_package(fmt REQUIRED)
set(CMAKE_PREFIX_PATH ${CMAKE_PREFIX_PATH_OLD})

include_directories(
    ${CMAKE_SOURCE_DIR}/src/Include
    #${CMAKE_SOURCE_DIR}/src/Include/Resource
    #${CMAKE_SOURCE_DIR}/src/Include/Exceptions
    #${CMAKE_CURRENT_BINARY_DIR}/Include
    #${CMAKE_CURRENT_BINARY_DIR}/Include/Resource
    SYSTEM ${LIB_DIR}
)

if(MSVC)
    add_compile_options(/W4 /WX)
    # Warning level 0 for all headers included by '<>'
    #add_compile_options(/experimental:external /external:anglebrackets /external:W0)
    #add_compile_options(/Zc:referenceBinding)
    add_compile_options(/wd4996)
endif()

if(UNIX)
    add_compile_options(-Wall -Wextra -Wno-reorder -Wno-unused-parameter
        -Werror -pedantic -pedantic-errors -Wno-error=array-bounds)
endif()

SET(BUILD_ALL TRUE CACHE BOOL TRUE)

option (BUILD_Server OFF)
option (BUILD_Client OFF)

if (BUILD_PROJECTS)
    set(BUILD_ALL FALSE)
    foreach(PROJECT ${BUILD_PROJECTS})
        set(BUILD_${PROJECT} TRUE)
    endforeach()
endif()

if (BUILD_ALL OR BUILD_Server)
        add_subdirectory(Server)
endif()
if (BUILD_ALL OR BUILD_Client)
        add_subdirectory(Client)
endif()