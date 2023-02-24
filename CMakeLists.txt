cmake_minimum_required (VERSION 3.0)

set(CMAKE_CXX_STANDARD 17)

if(POLICY CMP0012)
    cmake_policy(SET CMP0012 NEW)
endif()

#check out system type
if(WIN32)
    message("Windows version...")
    add_definitions(-DUNICODE -D_UNICODE)
elseif(UNIX)
    message("UNIX/Linux version...")
else()
    message(FATAL_ERROR "Unsupported platform")
endif()

set(LIB_DIR ${CMAKE_SOURCE_DIR}/libs)
set(EXTERNALS_INSTALL_DIR ${LIB_DIR})
set_property (GLOBAL PROPERTY USE_FOLDERS ON)



#list(APPEND CMAKE_MODULE_PATH ${LIB_DIR}/cmake)

set(OUTDIR_DEBUG "bin/x64")

file(MAKE_DIRECTORY ${CMAKE_SOURCE_DIR}/${OUTDIR_DEBUG})
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY_DEBUG ${CMAKE_SOURCE_DIR}/${OUTDIR_DEBUG})
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_DEBUG ${CMAKE_SOURCE_DIR}/${OUTDIR_DEBUG})
set(RUNTIME_OUTPUT_DIR ${CMAKE_SOURCE_DIR}/${OUTDIR_DEBUG})

if(WIN32)
    set(EXECUTABLES_EXT .exe)
else()
    set(EXECUTABLES_EXT)
endif()

if(UNIX)
    set(INSTALL_DIR /usr/bin/instal_dir)
else()
    set(INSTALL_DIR ${CMAKE_INSTALL_PREFIX})
endif()

set(INSTALL_COMPONENTS_DIR ${INSTALL_DIR}/Components)

add_subdirectory(src)

