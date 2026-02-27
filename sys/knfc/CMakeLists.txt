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

# CRT selection: default /MD (dynamic) to match ort-sys, whisper-rs-sys, etc.
# Set KNF_STATIC_CRT=1 env var to use /MT (static) instead.
# add_compile_options propagates to all subdirectories (including kaldi-native-fbank).
if(MSVC)
  if(CMAKE_MSVC_RUNTIME_LIBRARY MATCHES "MultiThreadedDLL" OR
     CMAKE_MSVC_RUNTIME_LIBRARY MATCHES "MultiThreadedDebugDLL" OR
     NOT DEFINED CMAKE_MSVC_RUNTIME_LIBRARY)
    add_compile_options(
        $<$<CONFIG:>:/MD>
        $<$<CONFIG:Debug>:/MDd>
        $<$<CONFIG:Release>:/MD>
        $<$<CONFIG:RelWithDebInfo>:/MD>
        $<$<CONFIG:MinSizeRel>:/MD>
    )
  else()
    add_compile_options(
        $<$<CONFIG:>:/MT>
        $<$<CONFIG:Debug>:/MTd>
        $<$<CONFIG:Release>:/MT>
        $<$<CONFIG:RelWithDebInfo>:/MT>
        $<$<CONFIG:MinSizeRel>:/MT>
    )
  endif()
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