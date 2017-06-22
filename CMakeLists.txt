cmake_minimum_required(VERSION 3.8)
project(parallel_programming_experiments)

set(CMAKE_CXX_STANDARD 17)

set(SOURCE_FILES main.cpp)
add_executable(parallel_programming_experiments ${SOURCE_FILES})

option(ppe_CLANG_TIDY "Configure project for Clang tidy" OFF)
if (ppe_CLANG_TIDY)
    # This creates a compile_commands.json in the build directory
    set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
    set(ppe_CLANG_TIDY_OUTPUT_DIR ${CMAKE_BINARY_DIR}/clang_tidy)
    file(MAKE_DIRECTORY ${ppe_CLANG_TIDY_OUTPUT_DIR})
endif()

if (ppe_CLANG_TIDY)

    # Find the executable; highest version number for preference
    find_program(
            CLANG_TIDY
            NAMES clang-tidy-5.0 clang-tidy-4.0 clang-tidy-3.9 clang-tidy-3.8
            HINTS $ENV{CLANG_TIDY_DIR} /usr/bin
            DOC "Clang tidy executable."
    )

    # Get the cpp files recursively from the source dir
    file(GLOB_RECURSE LIST_OF_TRANSLATION_UNITS RELATIVE ${CMAKE_SOURCE_DIR} *.cpp)

    # Add custom commands: one for each src file to process
    foreach(src_file ${LIST_OF_TRANSLATION_UNITS})
        # Generate output file name from source file path
        string(REPLACE "/" "_" out_file_name ${src_file})
        set(out_file "${CMAKE_BINARY_DIR}/clang_tidy/${out_file_name}")

        # Generate path to input file
        set(infile "${CMAKE_SOURCE_DIR}/${src_file}")

        # Custom command to do the processing
        ADD_CUSTOM_COMMAND(
                OUTPUT "${out_file}"
                COMMAND ${CLANG_TIDY} -p=${CMAKE_BINARY_DIR} ${CMAKE_SOURCE_DIR}/${src_file} > ${out_file}
                COMMENT "Producing clang_tidy report: ${out_file}"
        )

        # Finally set out_files for dependencies
        set(out_files ${out_files} "${out_file}")
    endforeach(src_file)

    # Create a target that will run clang_tidy on each target
    ADD_CUSTOM_TARGET(clang_tidy ALL DEPENDS ${out_files})
endif()