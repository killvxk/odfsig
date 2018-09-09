# Copyright 2018 Miklos Vajna. All rights reserved.
# Use of this source code is governed by a BSD-style license that can be
# found in the LICENSE file.

find_package(PkgConfig REQUIRED)

if (NOT ODFSIG_INTERNAL_XMLSEC)
    pkg_check_modules(XMLSEC REQUIRED xmlsec1-nss)
else ()
    include(ExternalProject)
    include(GNUInstallDirs)

    pkg_check_modules(NSS REQUIRED nss)
    pkg_check_modules(LIBXML REQUIRED libxml-2.0)

    ExternalProject_Add(extern-xmlsec-project
        URL http://www.aleksey.com/xmlsec/download/xmlsec1-1.2.26.tar.gz
        URL_HASH SHA256=8d8276c9c720ca42a3b0023df8b7ae41a2d6c5f9aa8d20ed1672d84cc8982d50
        DOWNLOAD_DIR ${CMAKE_SOURCE_DIR}/extern/tarballs
        BUILD_IN_SOURCE ON
        CONFIGURE_COMMAND ./configure
            --prefix=${CMAKE_CURRENT_BINARY_DIR}/install
            --with-pic
            --disable-shared
            --disable-crypto-dl
            --without-libxslt
            --without-gnutls
            --without-gcrypt
            --disable-apps
            --disable-docs
            --without-openssl
        BUILD_COMMAND make
        INSTALL_COMMAND make install
        )
    add_library(xmlsec1 STATIC IMPORTED)
    set_property(TARGET xmlsec1 PROPERTY IMPORTED_LOCATION ${CMAKE_CURRENT_BINARY_DIR}/install/${CMAKE_INSTALL_LIBDIR}/libxmlsec1.a)
    add_dependencies(xmlsec1 extern-xmlsec-project)
    add_library(xmlsec1-nss STATIC IMPORTED)
    set_property(TARGET xmlsec1-nss PROPERTY IMPORTED_LOCATION ${CMAKE_CURRENT_BINARY_DIR}/install/${CMAKE_INSTALL_LIBDIR}/libxmlsec1-nss.a)
    add_dependencies(xmlsec1-nss extern-xmlsec-project)

    set(XMLSEC_INCLUDE_DIRS
        ${NSS_INCLUDE_DIRS}
        ${LIBXML_INCLUDE_DIRS}
        ${CMAKE_CURRENT_BINARY_DIR}/install/include/xmlsec1
        )
    # nss after static libs to please --as-needed linkers.
    set(XMLSEC_LIBRARIES
        xmlsec1-nss
        xmlsec1
        ${NSS_LIBRARIES}
        ${LIBXML_LIBRARIES}
        )
    set(XMLSEC_CFLAGS_OTHER
        -DXMLSEC_NO_SIZE_T
        )
endif ()

add_library(libxmlsec INTERFACE)
target_include_directories(libxmlsec INTERFACE ${XMLSEC_INCLUDE_DIRS})
target_link_libraries(libxmlsec INTERFACE ${XMLSEC_LIBRARIES})
target_compile_options(libxmlsec INTERFACE ${XMLSEC_CFLAGS_OTHER})

# vim:set shiftwidth=4 softtabstop=4 expandtab: