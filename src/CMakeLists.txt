# Copyright 2018 Miklos Vajna. All rights reserved.
# Use of this source code is governed by a BSD-style license that can be
# found in the LICENSE file.

if (WIN32)
    set(CRYPTO cng)
    set(CRYPTO_LIBRARIES crypt32)
else ()
    set(CRYPTO nss)
endif()

add_library(odfsigcore
    crypto-${CRYPTO}.cxx
    lib.cxx
    main.cxx
    string.cxx
    )
target_include_directories(odfsigcore
    PUBLIC
    "${CMAKE_SOURCE_DIR}/include"
    "${CMAKE_BINARY_DIR}/include"
    )
target_link_libraries(odfsigcore
    libzip
    libxmlsec
    ${CRYPTO_LIBRARIES}
    )

# vim:set shiftwidth=4 softtabstop=4 expandtab:
