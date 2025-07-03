#
#     PADRING -- a padring generator for ASICs.
#
#     Copyright (c) 2019, Niels Moseley <niels@symbioticeda.com>
#
#     Permission to use, copy, modify, and/or distribute this software for any
#     purpose with or without fee is hereby granted, provided that the above
#     copyright notice and this permission notice appear in all copies.
#
#     THE SOFTWARE IS PROVIDED "AS IS" AND THE AUTHOR DISCLAIMS ALL WARRANTIES
#     WITH REGARD TO THIS SOFTWARE INCLUDING ALL IMPLIED WARRANTIES OF
#     MERCHANTABILITY AND FITNESS. IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR
#     ANY SPECIAL, DIRECT, INDIRECT, OR CONSEQUENTIAL DAMAGES OR ANY DAMAGES
#     WHATSOEVER RESULTING FROM LOSS OF USE, DATA OR PROFITS, WHETHER IN AN
#     ACTION OF CONTRACT, NEGLIGENCE OR OTHER TORTIOUS ACTION, ARISING OUT OF
#     OR IN CONNECTION WITH THE USE OR PERFORMANCE OF THIS SOFTWARE.
#
#
#
#
#

cmake_minimum_required(VERSION 3.10)
project(padring)

set (CMAKE_CXX_STANDARD 20)
set (CMAKE_BUILD_TYPE Debug)

add_definitions(-D_CRT_SECURE_NO_WARNINGS)

execute_process(
  COMMAND git log -1 --format=%h
  WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
  OUTPUT_VARIABLE GIT_COMMIT_HASH
  OUTPUT_STRIP_TRAILING_WHITESPACE
)

#-------------------------------------------------
# Doxygen
#-------------------------------------------------

option(BUILD_DOC "Build documentation" OFF)

# check if Doxygen is installed
find_package(Doxygen)
if (DOXYGEN_FOUND AND BUILD_DOC)
    # set input and output files
    set(DOXYGEN_IN  ${PROJECT_SOURCE_DIR}/doc/Doxyfile.in)
    set(DOXYGEN_OUT ${PROJECT_SOURCE_DIR}/doc/Doxyfile)

    # request to configure the file
    configure_file(${DOXYGEN_IN} ${DOXYGEN_OUT} @ONLY)
    message("Doxygen build started")

    # note the option ALL which allows to build the docs together with the application
    add_custom_target(doc_doxygen ALL
        COMMAND ${DOXYGEN_EXECUTABLE} ${DOXYGEN_OUT}
        WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}
        COMMENT "Generating API documentation with Doxygen"
        DEPENDS glnetlist
        VERBATIM )
else (DOXYGEN_FOUND AND BUILD_DOC)
  message("Doxygen need to be installed to generate the doxygen documentation")
endif (DOXYGEN_FOUND AND BUILD_DOC)

#-------------------------------------------------
# Libraries
#-------------------------------------------------
include(FetchContent)

FetchContent_Declare(spdlog
                     GIT_REPOSITORY https://github.com/gabime/spdlog.git
                     GIT_TAG        6fa36017cfd5731d617e1a934f0e5ea9c4445b13) # release-1.15.3


FetchContent_Declare(cxxopts
                     GIT_REPOSITORY https://github.com/jarro2783/cxxopts.git
                     GIT_TAG        44380e5a44706ab7347f400698c703eb2a196202) # release-3.3.1

# FetchContent_Declare(qhull
#                      GIT_REPOSITORY https://github.com/qhull/qhull.git
#                      GIT_TAG        613debeaea72ee66626dace9ba1a2eff11b5d37d) # release-3.3.1
# if (NOT qhull_POPULATED)
#   FetchContent_Populate(qhull)
#   set(BUILD_SHARED_LIBS OFF)
# endif()
#
# FetchContent_Declare(gdstk
#                      GIT_REPOSITORY https://github.com/heitzmann/gdstk.git
#                      GIT_TAG        f1df6544a88b851b96d26f72ab70852ff318deb4) # release-0.9.60

FetchContent_MakeAvailable(spdlog cxxopts) # qhull gdstk)

#-------------------------------------------------
# Padring
#-------------------------------------------------

include_directories("${CMAKE_CURRENT_SOURCE_DIR}/include/")

set(PADRING_SRCS
    ${PROJECT_SOURCE_DIR}/src/main.cpp
    ${PROJECT_SOURCE_DIR}/src/logging.cpp
    ${PROJECT_SOURCE_DIR}/src/layout.cpp
    ${PROJECT_SOURCE_DIR}/src/svgwriter.cpp
    ${PROJECT_SOURCE_DIR}/src/defwriter.cpp
    ${PROJECT_SOURCE_DIR}/src/prlefreader.cpp
    ${PROJECT_SOURCE_DIR}/src/configreader.cpp
    ${PROJECT_SOURCE_DIR}/src/lefreader.cpp
    ${PROJECT_SOURCE_DIR}/src/gds2writer.cpp
    ${PROJECT_SOURCE_DIR}/src/debugutils.cpp
)

add_executable(padring ${PADRING_SRCS})

target_include_directories(padring PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/include/")
target_link_libraries(padring PRIVATE spdlog cxxopts)
target_compile_definitions(padring PRIVATE __AUTHOR__="Daniel Schmeer" __PGMVERSION__="${GIT_COMMIT_HASH}")
