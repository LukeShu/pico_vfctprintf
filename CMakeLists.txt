# Copyright (c) 2020  Raspberry Pi (Trading) Ltd.
# SPDX-License-Identifier: BSD-3-Clause
#
# Copyright (c) 2025  Luke T. Shumaker
# SPDX-License-Identifier: BSD-3-Clause

if (NOT COMMAND "pico_add_library")
    cmake_minimum_required(VERSION 3.13...3.27)
    project(pico_vfctprintf)

    # create an INTERFACE library named target, and define LIB_TARGET=1 (upper case) as a compile option
    # and make it dependent on a pre-existing corresponding _headers library
    # optional arg NOFLAG will skip the LIB_TARGET definition
    function(pico_add_impl_library target)
        add_library(${target} INTERFACE)
        string(TOUPPER ${target} TARGET_UPPER)
        if (${ARGC} GREATER 1)
            if (NOT "${ARGV1}" STREQUAL "NOFLAG")
                message(FATAL_ERROR "Unknown parameter ${ARGV1}")
            endif()
        else()
            target_compile_definitions(${target} INTERFACE LIB_${TARGET_UPPER}=1)
        endif()
        target_link_libraries(${target} INTERFACE ${target}_headers)
    endfunction()

    # create an INTERFACE library named target along with associated header, and define LIB_TARGET=1 (upper case) as a compile option
    # optional arg NOFLAG will skip the LIB_TARGET definition
    function(pico_add_library target)
        add_library(${target}_headers INTERFACE)
        pico_add_impl_library(${target} ${ARGN})
    endfunction()

    set(PICO_SDK_TESTS_ENABLED 1)
endif()

if (NOT TARGET pico_vfctprintf)
    pico_add_library(pico_vfctprintf)

    target_include_directories(pico_vfctprintf_headers SYSTEM INTERFACE ${CMAKE_CURRENT_LIST_DIR}/include)

    target_sources(pico_vfctprintf INTERFACE
            ${CMAKE_CURRENT_LIST_DIR}/printf.c
    )
    target_link_libraries(pico_vfctprintf INTERFACE pico_vfctprintf_headers)

    if (PICO_SDK_TESTS_ENABLED)
        include(CTest)

        add_executable(test_suite test/test_suite.cpp)
        target_link_libraries(test_suite pico_vfctprintf)
        add_test(
            NAME    "pico_vfctprintf/test_suite"
            COMMAND valgrind --error-exitcode=2 ./test_suite
        )
    endif()
endif()
