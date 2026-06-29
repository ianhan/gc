cmake_minimum_required(VERSION 3.13)

project(gc
    VERSION 0.1.0
    LANGUAGES C
)

option(GC_ENABLE_DISPLAYLINK "Allow the aggregate gc target to include the DisplayLink backend when tinyusb_libdlo is available" ON)
set(GC_DEFAULT_TARGET "AUTO" CACHE STRING "Backend used by the aggregate gc/libgc target: AUTO, CORE, or DISPLAYLINK")
set_property(CACHE GC_DEFAULT_TARGET PROPERTY STRINGS AUTO CORE DISPLAYLINK)

add_library(gc_core INTERFACE)
target_sources(gc_core INTERFACE
    "${CMAKE_CURRENT_LIST_DIR}/src/gc.c"
    "${CMAKE_CURRENT_LIST_DIR}/src/gcattr.c"
    "${CMAKE_CURRENT_LIST_DIR}/src/gcmx.c"
    "${CMAKE_CURRENT_LIST_DIR}/src/gcrender.c"
    "${CMAKE_CURRENT_LIST_DIR}/src/gcsw.c"
)
target_include_directories(gc_core INTERFACE
    "${CMAKE_CURRENT_LIST_DIR}/src"
)
target_compile_features(gc_core INTERFACE
    c_std_11
)
target_link_libraries(gc_core INTERFACE
    m
)

add_library(gc::core ALIAS gc_core)

if(GC_ENABLE_DISPLAYLINK AND TARGET tinyusb_libdlo)
    add_library(gc_dlo INTERFACE)
    target_sources(gc_dlo INTERFACE
        "${CMAKE_CURRENT_LIST_DIR}/src/gcdlo.c"
    )
    target_link_libraries(gc_dlo INTERFACE
        gc_core
        tinyusb_libdlo
    )

    add_library(gc::displaylink ALIAS gc_dlo)
    add_library(libgc_dlo ALIAS gc_dlo)
endif()

string(TOUPPER "${GC_DEFAULT_TARGET}" GC_DEFAULT_TARGET_UC)
if(NOT GC_DEFAULT_TARGET_UC MATCHES "^(AUTO|CORE|DISPLAYLINK)$")
    message(FATAL_ERROR "GC_DEFAULT_TARGET must be AUTO, CORE, or DISPLAYLINK")
endif()

set(GC_AGGREGATE_TARGET gc_core)
if(GC_DEFAULT_TARGET_UC STREQUAL "DISPLAYLINK")
    if(NOT TARGET gc_dlo)
        message(FATAL_ERROR "GC_DEFAULT_TARGET=DISPLAYLINK requires GC_ENABLE_DISPLAYLINK=ON and the tinyusb_libdlo target")
    endif()
    set(GC_AGGREGATE_TARGET gc_dlo)
elseif(GC_DEFAULT_TARGET_UC STREQUAL "AUTO" AND TARGET gc_dlo)
    set(GC_AGGREGATE_TARGET gc_dlo)
endif()

add_library(gc INTERFACE)
target_link_libraries(gc INTERFACE
    "${GC_AGGREGATE_TARGET}"
)

add_library(libgc ALIAS gc)
add_library(gc::gc ALIAS gc)
