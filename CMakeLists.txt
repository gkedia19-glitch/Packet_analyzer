cmake_minimum_required(VERSION 3.16)
project(PacketAnalyzer VERSION 2.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Default to an optimized build when none is specified
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    set(CMAKE_BUILD_TYPE Release)
endif()

find_package(Threads REQUIRED)

# Shared core: PCAP reading, packet parsing, SNI extraction, app classification
add_library(dpi_core STATIC
    src/pcap_reader.cpp
    src/packet_parser.cpp
    src/sni_extractor.cpp
    src/types.cpp
)
target_include_directories(dpi_core PUBLIC ${CMAKE_SOURCE_DIR}/include)

# Multi-threaded DPI engine (main program)
add_executable(dpi_engine src/dpi_mt.cpp)
target_link_libraries(dpi_engine PRIVATE dpi_core Threads::Threads)

# Simple single-threaded version (for learning and small captures)
add_executable(dpi_simple src/main_working.cpp)
target_link_libraries(dpi_simple PRIVATE dpi_core)

if(MSVC)
    target_compile_options(dpi_core PRIVATE /W3)
else()
    target_compile_options(dpi_core PRIVATE -Wall -Wextra)
endif()
