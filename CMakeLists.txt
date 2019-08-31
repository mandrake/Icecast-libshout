cmake_minimum_required(VERSION 3.2)

project(shout VERSION 1.0 LANGUAGES C CXX)

set(SOURCES
    src/connection.c
    src/format_mp3.c
    src/format_webm.c
    src/proto_http.c
    src/proto_icy.c
    src/proto_roaraudio.c
    src/proto_xaudiocast.c
    src/queue.c
    src/shout_private.h
    src/shout.c
    src/tls.c
    src/util.c
    src/util.h)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC")

if(NOT DEFINED SHOUT_THREADSAFE)
    message("SHOUT_THREADSAFE not specified, defaulting to 1")
    set(SHOUT_THREADSAFE "1")
endif()

if(NOT DEFINED SHOUT_TLS)
    message("SHOUT_TLS not specified, defaulting to 1")
    set(SHOUT_TLS "1")
endif()

if(NOT DEFINED SHOUT_OGG)
    message("SHOUT_OGG not specified, defaulting to 0")
    set(SHOUT_OGG "0")
endif()

if(NOT DEFINED SHOUT_THEORA)
    message("SHOUT_THEORA not specified, defaulting to 0")
    set(SHOUT_THEORA "0")
endif()

if(NOT DEFINED SHOUT_VORBIS)
    message("SHOUT_VORBIS not specified, defaulting to 0")
    set(SHOUT_VORBIS "0")
endif()

if(NOT DEFINED SHOUT_SPEEX)
    message("SHOUT_SPEEX not specified, defaulting to 0")
    set(SHOUT_SPEEX "0")
endif()

add_definitions(
    -DSHOUT_THREADSAFE=${SHOUT_THREADSAFE}
    -DSHOUT_TLS=${SHOUT_TLS}
    -DSHOUT_OGG=${SHOUT_OGG}
    -DHAVE_STDINT_H
    -DHAVE_GETTIMEOFDAY
    -DHAVE_SYS_STAT_H
    -DTIME_WITH_SYSTIME
    -DHAVE_SYS_TIME_H
    -DLIBSHOUT_MAJOR=2
    -DLIBSHOUT_MINOR=0
    -DLIBSHOUT_MICRO=0
    -DVERSION="2.0.0")

add_subdirectory(src/common/avl)
add_subdirectory(src/common/httpp)
add_subdirectory(src/common/log)
add_subdirectory(src/common/net)
add_subdirectory(src/common/thread)
add_subdirectory(src/common/timing)

add_library(shout SHARED ${SOURCES})
target_include_directories(shout PRIVATE
    include
    src
    src/common)
target_link_libraries(shout avl httpp log net thread timing pthread)

if(SHOUT_TLS EQUAL 1)
    set(SOURCES ${SOURCES} src/tls.c)
    add_definitions(-DHAVE_OPENSSL)
    find_package(OpenSSL REQUIRED)
    target_link_libraries(shout ssl crypto)
    # TODO: linking to openssl doesn't work, fix
endif()

if(SHOUT_OGG EQUAL 1)
    set(SOURCES ${SOURCES} src/format_ogg.c src/format_ogg.h src/codec_opus.h)
    # TODO: find package ogg and link to it
endif()

if(SHOUT_THEORA EQUAL 1)
    set(SOURCES ${SOURCES} src/codec_theora.c)
    # TODO: find package theora and link to it
endif()

if(SHOUT_VORBIS EQUAL 1)
    set(SOURCES ${SOURCES} src/codec_vorbis.c)
    # TODO: find package vorbis and link to it
endif()

if(SHOUT_SPEEX EQUAL 1)
    set(SOURCES ${SOURCES} src/codec_speex.c)
    # TODO: find package speex and link to it
endif()