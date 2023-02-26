#  Input variables:
#   EXTERNALS_INSTALL_DIR
#   CMAKE_PREFIX_PATH
#   LIB_DIR

option (REBUILD_EXTERNAL_LIBS "Rebuild external libraries" OFF)

if("$ENV{REBUILD_EXTERNAL_LIBS}" STREQUAL "true")
    set(REBUILD_EXTERNAL_LIBS ON)
endif()

if(MSVC)
    set(TEMP_DIR ${CMAKE_BINARY_DIR}/tempbuild)
    if(NOT EXISTS "${CMAKE_BINARY_DIR}/temp")
        file(MAKE_DIRECTORY ${TEMP_DIR})
         message("MAKE_DIRECTORY BUILD DIR IS: ${CMAKE_BINARY_DIR}/temp")
    endif()
    set(EXTERNALS_BUILD_DIR ${TEMP_DIR})
     message("EXTERNAL BUILD DIR IS: $ENV{TEMP}/project_Hellgate")
    #set(EXTERNALS_BUILD_DIR $ENV{TEMP}/project_Hellgate)
    #message("EXTERNAL BUILD DIR IS: $ENV{TEMP}/project_Hellgate")
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

#OpenSSL

set(OPENSSL_VERSION_MAJOR "1")
set(OPENSSL_VERSION_MINOR "0")
set(OPENSSL_VERSION_PATCH "2u")
set(OPENSSL_DIR_NAME openssl-${OPENSSL_VERSION_MAJOR}.${OPENSSL_VERSION_MINOR}.${OPENSSL_VERSION_PATCH})
set(OPENSSL_INSTALL_DIR ${EXTERNALS_INSTALL_DIR}/${OPENSSL_DIR_NAME})

if(REBUILD_EXTERNAL_LIBS)
    file(REMOVE_RECURSE ${OPENSSL_INSTALL_DIR})
endif()

if(NOT EXISTS "${OPENSSL_INSTALL_DIR}/include/openssl/ssl.h")
    set(OPENSSL_BUILD_DIR ${EXTERNALS_BUILD_DIR}/${OPENSSL_DIR_NAME})
    file(REMOVE_RECURSE ${OPENSSL_BUILD_DIR})
    file(MAKE_DIRECTORY ${OPENSSL_BUILD_DIR})
    execute_process(
        COMMAND ${CMAKE_COMMAND}
            -DOPENSSL_VERSION_MAJOR=${OPENSSL_VERSION_MAJOR}
            -DOPENSSL_VERSION_MINOR=${OPENSSL_VERSION_MINOR}
            -DOPENSSL_VERSION_PATCH=${OPENSSL_VERSION_PATCH}
            -DOPENSSL_INSTALL_DIR=${OPENSSL_INSTALL_DIR}
            -DOPENSSL_BINARIES_DIR_DEBUG=${CMAKE_SOURCE_DIR}/${OUTDIR_DEBUG}
            -DOPENSSL_BINARIES_DIR_RELEASE=${CMAKE_SOURCE_DIR}/${OUTDIR_RELEASE}
            -G ${CMAKE_GENERATOR}
            ${CMAKE_GENERATOR_PLATFORM}
            -S ${CMAKE_SOURCE_DIR}/external/OpenSSL
            -B ${OPENSSL_BUILD_DIR}
        WORKING_DIRECTORY ${OPENSSL_BUILD_DIR}
    )
    execute_process(
        COMMAND ${CMAKE_COMMAND}
            --build .
        WORKING_DIRECTORY ${OPENSSL_BUILD_DIR}
    )
endif()

#Hints
set(OPENSSL_ROOT_DIR ${OPENSSL_INSTALL_DIR})

find_package(OpenSSL REQUIRED)

#Protobuf

set(PROTOBUF_GIT_TAG "v3.13.0")
set(PROTOBUF_DIR_NAME protobuf-${PROTOBUF_GIT_TAG})
set(PROTOBUF_INSTALL_DIR ${EXTERNALS_INSTALL_DIR}/${PROTOBUF_DIR_NAME})

if(NOT EXISTS "${PROTOBUF_INSTALL_DIR}")
    set(PROTOBUF_BUILD_DIRECTORY ${EXTERNALS_BUILD_DIR}/external/${PROTOBUF_DIR_NAME})
    file(REMOVE_RECURSE ${PROTOBUF_BUILD_DIRECTORY})
    file(MAKE_DIRECTORY ${PROTOBUF_BUILD_DIRECTORY})
    execute_process(
        COMMAND ${CMAKE_COMMAND}
            -DPROTOBUF_GIT_TAG=${PROTOBUF_GIT_TAG}
            -DPROTOBUF_BUILD_DIR=${PROTOBUF_BUILD_DIRECTORY}
            -DPROTOBUF_INSTALL_DIR=${PROTOBUF_INSTALL_DIR}
            -G ${CMAKE_GENERATOR}
            ${CMAKE_GENERATOR_PLATFORM}
            -S ${CMAKE_SOURCE_DIR}/external/protobuf
            -B ${PROTOBUF_BUILD_DIRECTORY}
        WORKING_DIRECTORY ${PROTOBUF_BUILD_DIRECTORY}
    )
    execute_process(
        COMMAND ${CMAKE_COMMAND}
            --build .
        WORKING_DIRECTORY ${PROTOBUF_BUILD_DIRECTORY}
    )

endif()

find_package(Protobuf CONFIG REQUIRED PATHS ${PROTOBUF_INSTALL_DIR} NO_DEFAULT_PATH)

#GRPC

set(GRPC_GIT_TAG "v1.34.0")
set(GRPC_DIR_NAME grpc-${GRPC_GIT_TAG})
set(GRPC_INSTALL_DIR ${EXTERNALS_INSTALL_DIR}/${GRPC_DIR_NAME})

if(NOT EXISTS "${GRPC_INSTALL_DIR}")
    set(GRPC_BUILD_DIRECTORY ${EXTERNALS_BUILD_DIR}/external/${GRPC_DIR_NAME})
    file(REMOVE_RECURSE ${GRPC_BUILD_DIRECTORY})
    file(MAKE_DIRECTORY ${GRPC_BUILD_DIRECTORY})
    execute_process(
        COMMAND ${CMAKE_COMMAND}
            -DGRPC_GIT_TAG=${GRPC_GIT_TAG}
            -DGRPC_BUILD_DIR=${GRPC_BUILD_DIRECTORY}
            -DGRPC_INSTALL_DIR=${GRPC_INSTALL_DIR}
            -DGRPC_PROTOBUF_ROOT_DIR=${Protobuf_DIR}
            -DGRPC_OPENSSL_ROOT_DIR=${OPENSSL_INSTALL_DIR}
            -G ${CMAKE_GENERATOR}
            ${CMAKE_GENERATOR_PLATFORM}
            -S ${CMAKE_SOURCE_DIR}/external/grpc
            -B ${GRPC_BUILD_DIRECTORY}
        WORKING_DIRECTORY ${GRPC_BUILD_DIRECTORY}
    )
    execute_process(
        COMMAND ${CMAKE_COMMAND}
            --build .
        WORKING_DIRECTORY ${GRPC_BUILD_DIRECTORY}
    )
endif()

find_package(gRPC CONFIG REQUIRED PATHS ${GRPC_INSTALL_DIR} NO_DEFAULT_PATH)


include_directories(
    ${CMAKE_SOURCE_DIR}/src/Include
    #${CMAKE_SOURCE_DIR}/src/Include/Resource
    #${CMAKE_SOURCE_DIR}/src/Include/Exceptions
    ${CMAKE_CURRENT_BINARY_DIR}/Include
    SYSTEM ${LIB_DIR}
)

if(MSVC)
    add_compile_options(/W4 /WX)
    # Warning level 0 for all headers included by '<>'
    add_compile_options(/experimental:external /external:anglebrackets /external:W0)
    add_compile_options(/Zc:referenceBinding)
    add_compile_options(/wd4996)
endif()

if(UNIX)
    add_compile_options(-Wall -Wextra -Wno-reorder -Wno-unused-parameter
        -Werror -pedantic -pedantic-errors -Wno-error=array-bounds)
endif()

if(MSVC)
    add_definitions(-D_SILENCE_CXX17_ALLOCATOR_VOID_DEPRECATION_WARNING)
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