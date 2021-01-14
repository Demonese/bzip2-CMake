cmake_minimum_required(VERSION 3.0.0)
project(bzip2)

if(MSVC)
    # I known, shut up please...
    add_compile_definitions(
        _CRT_SECURE_NO_WARNINGS
        _CRT_NONSTDC_NO_WARNINGS
    )
endif()

# basic sources

set(BZ2_SRC_ROOT src) # remember to modify it!
set(BZ2_SRC
    ${BZ2_SRC_ROOT}/blocksort.c
    ${BZ2_SRC_ROOT}/bzlib.h
    ${BZ2_SRC_ROOT}/bzlib_private.h
    ${BZ2_SRC_ROOT}/bzlib.c
    ${BZ2_SRC_ROOT}/compress.c
    ${BZ2_SRC_ROOT}/crctable.c
    ${BZ2_SRC_ROOT}/decompress.c
    ${BZ2_SRC_ROOT}/huffman.c
    ${BZ2_SRC_ROOT}/randtable.c
)

# static library

add_library(libbz2_static STATIC)
target_compile_definitions(libbz2_static PUBLIC
    _FILE_OFFSET_BITS=64
)
target_include_directories(libbz2_static PUBLIC
    ${BZ2_SRC_ROOT}
)
target_sources(libbz2_static PRIVATE
    ${BZ2_SRC}
)

# dynamic library

add_library(libbz2 SHARED)
target_compile_definitions(libbz2 PUBLIC
    _FILE_OFFSET_BITS=64
)
target_include_directories(libbz2 PUBLIC
    ${BZ2_SRC_ROOT}
)
target_sources(libbz2 PRIVATE
    ${BZ2_SRC}
)
if(MSVC)
    # NOTE: might recive a warning LNK4017 but WHO care :)
    target_sources(libbz2 PRIVATE
        ${BZ2_SRC_ROOT}/libbz2.def
    )
endif()

# bzip2

set(_BZIP2_CLI_EXECUTABLES
    bzip2
    bunzip2
    bzcat
)
foreach(target_name ${_BZIP2_CLI_EXECUTABLES})
    add_executable(${target_name})
    target_sources(${target_name} PRIVATE
        ${BZ2_SRC_ROOT}/bzip2.c
    )
    target_link_libraries(${target_name} PRIVATE
        libbz2_static
    )
endforeach()

# bzip2recover

add_executable(bzip2recover)
target_sources(bzip2recover PRIVATE
    ${BZ2_SRC_ROOT}/bzip2recover.c
)
target_link_libraries(bzip2recover PRIVATE
    libbz2_static
)

# install

include(GNUInstallDirs)
install(TARGETS libbz2_static libbz2 ${_BZIP2_CLI_EXECUTABLES} bzip2recover
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
)
install(FILES
    ${BZ2_SRC_ROOT}/bzlib.h DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)
