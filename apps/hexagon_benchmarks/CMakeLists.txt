cmake_minimum_required(VERSION 3.22)
project(hexagon_benchmark)

enable_testing()

# Set up language settings
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED YES)
set(CMAKE_CXX_EXTENSIONS NO)

# Find Halide
find_package(Halide REQUIRED)
find_package(OpenMP)

# Generator
add_halide_generator(conv.generator SOURCES conv3x3_generator.cpp)
add_halide_generator(dilate.generator SOURCES dilate3x3_generator.cpp)
add_halide_generator(gaussian.generator SOURCES gaussian5x5_generator.cpp)
add_halide_generator(median.generator SOURCES median3x3_generator.cpp)
add_halide_generator(sobel.generator SOURCES sobel_generator.cpp)

# Filters
add_halide_library(conv3x3a16 FROM conv.generator GENERATOR conv3x3 PARAMS accumulator_type=int16)
add_halide_library(dilate3x3 FROM dilate.generator)
add_halide_library(gaussian5x5 FROM gaussian.generator)
add_halide_library(median3x3 FROM median.generator)
add_halide_library(sobel FROM sobel.generator)
add_halide_library(conv3x3a32 FROM conv.generator GENERATOR conv3x3 PARAMS accumulator_type=int32 FEATURES hvx-hvx_v65)

# Enabling tests
# ( Removing the macro from the below which you don't want to use )
add_compile_definitions(CONV3X3A16 SOBEL DILATE3X3 MEDIAN3X3 GAUSSIAN5X5 CONV3X3A32)

# Main executable
add_executable(hexagon_benchmark_test process.cpp)
target_compile_options(hexagon_benchmark_test PRIVATE $<$<CXX_COMPILER_ID:GNU,Clang,AppleClang>:-O2>)
target_link_libraries(hexagon_benchmark_test
                      PRIVATE
                      Halide::Tools
                      conv3x3a16
                      dilate3x3
                      gaussian5x5
                      median3x3
                      sobel
                      conv3x3a32
                      $<TARGET_NAME_IF_EXISTS:OpenMP::OpenMP_CXX>)

# Test that the app actually works!
add_test(NAME hexagon_benchmark_app COMMAND hexagon_benchmark_test)
set_tests_properties(hexagon_benchmark_app PROPERTIES
                     LABELS hexagon_benchmark
                     PASS_REGULAR_EXPRESSION "Success!"
                     SKIP_REGULAR_EXPRESSION "\\[SKIP\\]")
