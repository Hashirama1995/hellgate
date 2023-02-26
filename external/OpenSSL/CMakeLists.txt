#   Input variables:
#       OPENSSL_VERSION_MAJOR
#       OPENSSL_VERSION_MINOR
#       OPENSSL_VERSION_PATCH
#       OPENSSL_INSTALL_DIR
#       OPENSSL_BINARIES_DIR_DEBUG
#       OPENSSL_BINARIES_DIR_RELEASE

cmake_minimum_required (VERSION 3.0)

project(OpenSSLBuild)

include(ExternalProject)

set(OPENSSL_URL "https://github.com/openssl/openssl/archive/OpenSSL_${OPENSSL_VERSION_MAJOR}_${OPENSSL_VERSION_MINOR}_${OPENSSL_VERSION_PATCH}.tar.gz")

if(MSVC)
    set(OPENSSL_CONFIGURE_COMMAND perl Configure VC-WIN64A "--prefix=${OPENSSL_INSTALL_DIR}")
    set(OPENSSL_BUILD_COMMAND "ms\\do_win64a.bat")
    set(OPENSSL_COMMAND nmake -f "ms\\ntdll.mak")
    set(OPENSSL_INSTALL_COMMAND nmake -f "ms\\ntdll.mak" install)
else()
    set(OPENSSL_CONFIGURE_COMMAND ./Configure linux-x86_64 no-shared no-zlib no-asm -fPIC --prefix=${OPENSSL_INSTALL_DIR})
    set(OPENSSL_BUILD_COMMAND make)
    set(OPENSSL_COMMAND "")
    set(OPENSSL_INSTALL_COMMAND make install)
endif()

ExternalProject_Add(
    OpenSSL
    URL ${OPENSSL_URL}
    CONFIGURE_COMMAND ${OPENSSL_CONFIGURE_COMMAND}
    BUILD_COMMAND ${OPENSSL_BUILD_COMMAND}
    COMMAND ${OPENSSL_COMMAND}
    BUILD_IN_SOURCE 1
    INSTALL_COMMAND ${OPENSSL_INSTALL_COMMAND}
)

if(MSVC)
    file(MAKE_DIRECTORY ${OPENSSL_BINARIES_DIR_DEBUG})
    file(MAKE_DIRECTORY ${OPENSSL_BINARIES_DIR_RELEASE})
    ExternalProject_Add_Step(
        OpenSSL OpenSSLCopyToBin
        COMMAND ${CMAKE_COMMAND} -E copy ${OPENSSL_INSTALL_DIR}/bin/libeay32.dll ${OPENSSL_BINARIES_DIR_RELEASE}
        COMMAND ${CMAKE_COMMAND} -E copy ${OPENSSL_INSTALL_DIR}/bin/libeay32.dll ${OPENSSL_BINARIES_DIR_DEBUG}
        COMMAND ${CMAKE_COMMAND} -E copy ${OPENSSL_INSTALL_DIR}/bin/ssleay32.dll ${OPENSSL_BINARIES_DIR_RELEASE}
        COMMAND ${CMAKE_COMMAND} -E copy ${OPENSSL_INSTALL_DIR}/bin/ssleay32.dll ${OPENSSL_BINARIES_DIR_DEBUG}
        DEPENDEES install
    )
endif()
