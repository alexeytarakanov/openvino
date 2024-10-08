# Copyright (C) 2018-2024 Intel Corporation
# SPDX-License-Identifier: Apache-2.0
#

set(BUILD_SHARED_LIBS OFF)

#
# zeroApi
#

if(ENABLE_ZEROAPI_BACKEND)
    # We have to avoid linking against loader with debug postfix due it's a part of driver
    # and the name will be the same for release and debug configurations
    set(CMAKE_DEBUG_POSTFIX "")

    # Skip warnings as errors for thirdparty
    if(CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
        ov_add_compiler_flags(/WX-)
        # solve pdb access issue
        set(USE_Z7 ON)
        # Close spectre for ze loader
        add_compile_options("/Qspectre-")
    elseif(CMAKE_COMPILER_IS_GNUCXX OR OV_COMPILER_IS_CLANG)
        ov_add_compiler_flags(-Wno-error)
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} \
            -Wno-undef \
            -Wno-missing-declarations")
        if(UNUSED_BUT_SET_VARIABLE_SUPPORTED)
            set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-unused-but-set-variable")
        endif()
    endif()
    set(CMAKE_COMPILE_WARNING_AS_ERROR OFF)
    add_subdirectory(level-zero EXCLUDE_FROM_ALL)

    # This VERSION file created by L0 may cause compilation issue of oneTBB headers, so remove it
    file(REMOVE "${CMAKE_BINARY_DIR}/VERSION")
endif()
