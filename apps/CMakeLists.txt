##
# Test apps from the perspective of a consuming project.
##

cmake_minimum_required(VERSION 3.22)
project(Halide_apps)

enable_testing()

if (WIN32)
    option(ENABLE_APPS_HANNK "Build apps/hannk" OFF)
else ()
    option(ENABLE_APPS_HANNK "Build apps/hannk" ON)
endif ()

function(add_app app_name)
    string(TOUPPER "ENABLE_APPS_${app_name}" opt)
    option(${opt} "Build apps/${app_name}" ON)
    if (${opt})
        add_subdirectory(${app_name})
    endif ()
endfunction()

# TODO: most of the apps need to be smartened to be crosscompilable under wasm.
message(STATUS "Halide_TARGET ${Halide_TARGET}")
if (Halide_TARGET MATCHES "wasm")
    message(WARNING "Skipping apps when building under wasm")
    return()
endif()

add_app(blur)
add_app(camera_pipe)
add_app(hexagon_benchmarks)  # TODO(#5374): missing CMake build
# add_app(hexagon_dma)  # TODO(#5374): missing CMake build