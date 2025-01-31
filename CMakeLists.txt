cmake_minimum_required(VERSION 3.16)

project(msdfgen LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

option(BUILD_SHARED_LIBS "Build shared instead of static library" OFF)

add_library(msdfgen STATIC)

set_target_properties(msdfgen PROPERTIES
    ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin/${CMAKE_BUILD_TYPE}"
    LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin/${CMAKE_BUILD_TYPE}"
    RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin/${CMAKE_BUILD_TYPE}"
)

file(GLOB_RECURSE MSDFGEN_CORE_SRC "core/*.cpp" "core/*.h" "core/*.hpp")
file(GLOB_RECURSE MSDFGEN_EXT_SRC "ext/*.cpp" "ext/*.h" "ext/*.hpp")
set(MSDFGEN_HEADERS "msdfgen.h" "msdfgen-ext.h" "resource.h")

target_sources(msdfgen PRIVATE ${MSDFGEN_CORE_SRC} ${MSDFGEN_EXT_SRC} ${MSDFGEN_HEADERS})

target_include_directories(msdfgen PRIVATE "include")

set(FT_DISABLE_HARFBUZZ ON)
set(FT_DISABLE_ZLIB ON)
set(FT_DISABLE_BZIP2 ON)
set(FT_DISABLE_PNG ON)
set(FT_DISABLE_BROTLI ON)

add_subdirectory(freetype)
target_include_directories(msdfgen PRIVATE freetype/include)
target_link_libraries(msdfgen PRIVATE freetype)

target_compile_definitions(msdfgen PRIVATE MSDFGEN_USE_CPP11)

target_link_libraries(msdfgen PRIVATE freetype)

if(WIN32)
    target_compile_definitions(msdfgen PRIVATE _WINDOWS)
endif()

set(CMAKE_DEBUG_POSTFIX "d")

if(CMAKE_BUILD_TYPE STREQUAL "Debug")
    target_compile_definitions(msdfgen PRIVATE DEBUG)
    target_compile_options(msdfgen PRIVATE -g)
elseif(CMAKE_BUILD_TYPE STREQUAL "Release" OR CMAKE_BUILD_TYPE STREQUAL "Dist")
    target_compile_definitions(msdfgen PRIVATE NDEBUG)
    target_compile_options(msdfgen PRIVATE -O2)
endif()
