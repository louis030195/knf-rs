cmake_minimum_required(VERSION 3.15)

# Enable CMAKE_MSVC_RUNTIME_LIBRARY support (set by cmake crate's static_crt()).
# CMP0091 must be NEW for CMAKE_MSVC_RUNTIME_LIBRARY to work.
# CMAKE_POLICY_DEFAULT_CMP0091 propagates to subdirectories that call
# cmake_minimum_required() with a version < 3.15.
cmake_policy(SET CMP0091 NEW)
set(CMAKE_POLICY_DEFAULT_CMP0091 NEW CACHE STRING "" FORCE)

project(knfc)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED True)

option(BUILD_SHARED_LIBS "Build shared libraries" OFF)
set(KALDI_NATIVE_FBANK_BUILD_PYTHON OFF CACHE BOOL "Disable Python build")
set(KALDI_NATIVE_FBANK_BUILD_TESTS OFF CACHE BOOL "Disable Tests build")

# Use /Zl to omit default library names and /FAILIFMISMATCH directives from .obj files.
# This prevents CRT mismatch errors (LNK2038) when linked with other crates that may
# use a different CRT (e.g., pre-built onnxruntime.lib uses /MD while whisper-rs uses /MT).
# The final binary's CRT is determined by the consuming project, not by this library.
if(MSVC)
  add_compile_options(/Zl)
endif()

add_subdirectory(../knf ${CMAKE_BINARY_DIR}/knf)

add_library(knfc STATIC knfc.cc)

target_link_libraries(knfc PRIVATE kaldi-native-fbank-core)

target_include_directories(knfc PRIVATE
    ${KNF_LIB_PATH}/include
    ../knf/kaldi-native-fbank/csrc
    ../knf
)

install(TARGETS knfc
        ARCHIVE DESTINATION lib)