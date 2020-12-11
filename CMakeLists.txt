cmake_minimum_required(VERSION 3.4)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

option(BUILD_COVERAGE "Build coverage" OFF)

set(
  HUNTER_CACHE_SERVERS
  "https://github.com/bmstu-iu8-cpp-sem-3/hunter-cache"
  CACHE STRING "Default cache server"
)

include("tools/gate/cmake/HunterGate.cmake")

HunterGate(
  URL "https://github.com/cpp-pm/hunter/archive/v0.23.278.tar.gz"
  SHA1 "35fa55bc1dfcf9f4d7264f8bb26cd1759ce1ae07"
)

project(lab-04)
string(APPEND CMAKE_CXX_FLAGS " -pedantic -Werror -Wall -Wextra")
string(APPEND CMAKE_CXX_FLAGS " -Wno-unused-command-line-argument")
string(APPEND CMAKE_CXX_FLAGS " -Wshadow -Wnon-virtual-dtor")

hunter_add_package(GTest)
find_package(GTest CONFIG REQUIRED)
hunter_add_package(Boost COMPONENTS filesystem)
find_package(Boost CONFIG REQUIRED filesystem)

add_library(lab-04 STATIC
        ${CMAKE_CURRENT_SOURCE_DIR}/sources/File.cpp
        )
add_executable(main
        ${CMAKE_CURRENT_SOURCE_DIR}/sources/main_File.cpp
        )

add_executable(tests
        ${CMAKE_CURRENT_SOURCE_DIR}/tests/File_test.cpp
        )

target_include_directories(lab-04
        PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include
        )
target_include_directories(main
        PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include
        )
target_include_directories(tests
        PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include
        )
target_link_libraries(tests GTest::gtest_main lab-04)
target_link_libraries(lab-04 Boost::filesystem)
target_link_libraries(main lab-04)

file(COPY misc DESTINATION ${CMAKE_CURRENT_BINARY_DIR})

enable_testing()
add_test(NAME unit_tests COMMAND tests)

if(BUILD_COVERAGE)
    function(ENABLE_PROJECT_COVERAGE PROJECT)
        target_compile_options(${PROJECT} PRIVATE --coverage)
        target_link_libraries(${PROJECT} --coverage)
        add_coverage(${PROJECT})
    endfunction()

    set(ENABLE_COVERAGE ON CACHE BOOL "Enable coverage build." FORCE)
    list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
    find_package(codecov)
    enable_project_coverage(main)
    enable_project_coverage(tests)
    list(APPEND LCOV_REMOVE_PATTERNS "'${PROJECT_SOURCE_DIR}/tests/*'")
    coverage_evaluate()
endif()
