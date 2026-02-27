cmake_minimum_required(VERSION 3.15)

# Enable CMAKE_MSVC_RUNTIME_LIBRARY support (set by cmake crate's static_crt())
cmake_policy(SET CMP0091 NEW)

project(knfc)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED True)

option(BUILD_SHARED_LIBS "Build shared libraries" OFF)
set(KALDI_NATIVE_FBANK_BUILD_PYTHON OFF CACHE BOOL "Disable Python build")
set(KALDI_NATIVE_FBANK_BUILD_TESTS OFF CACHE BOOL "Disable Tests build")

# CRT selection is handled by CMAKE_MSVC_RUNTIME_LIBRARY (passed by cmake crate).
# Default is /MD (dynamic CRT) to match ort-sys, whisper-rs-sys, etc.
# Set KNF_STATIC_CRT=1 env var to use /MT (static CRT) instead.

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