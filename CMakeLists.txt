cmake_minimum_required(VERSION 3.4)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

option(BUILD_TESTS "Build tests" ON)
option(BUILD_DOCS "Build documentation" OFF)
option(BUILD_COVERAGE "Build code coverage" OFF)

set(
        HUNTER_CACHE_SERVERS
        "https://github.com/bmstu-iu8-cpp-sem-3/hunter-cache"
        CACHE STRING "Default cache server"
)

include("cmake/HunterGate.cmake")

HunterGate(
        URL "https://github.com/cpp-pm/hunter/archive/v0.23.278.tar.gz"
        SHA1 "35fa55bc1dfcf9f4d7264f8bb26cd1759ce1ae07"
)

project(builder)
hunter_add_package(GTest)
find_package(GTest CONFIG REQUIRED)
hunter_add_package(Boost COMPONENTS program_options filesystem system regex)
hunter_add_package(Async++)
find_package(Async++ CONFIG REQUIRED)
find_package(Boost CONFIG REQUIRED program_options filesystem system regex)

add_library(builder STATIC
        ${CMAKE_CURRENT_SOURCE_DIR}/sources/my_cmake.cpp
        )
add_library(process INTERFACE
        )
add_executable(main
        ${CMAKE_CURRENT_SOURCE_DIR}/builder/builder.cpp
        )

target_include_directories(process
        INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/process/include
        )
target_include_directories(builder
        PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include
        )

target_link_libraries(builder process Async++::Async++
        Boost::program_options Boost::filesystem Boost::system Boost::regex)

add_executable(tests

        ${CMAKE_CURRENT_SOURCE_DIR}/tests/test.cpp
        )

target_link_libraries(
        tests
        builder
        GTest::gtest_main
)
enable_testing()
add_test(NAME unit_tests COMMAND tests)
target_link_libraries(main builder)

if(BUILD_COVERAGE)
    set(ENABLE_COVERAGE ON CACHE BOOL "Enable coverage build." FORCE)
    list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/tools/coverage/cmake")
    find_package(codecov)
    string(APPEND CMAKE_CXX_FLAGS " --coverage")
    add_coverage(${PROJECT_NAME})
    add_coverage(tests)
    coverage_evaluate()
endif()
