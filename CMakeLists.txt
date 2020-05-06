cmake_minimum_required(VERSION 3.10)

include("cmake/HunterGate.cmake")
HunterGate(
    URL "https:\//github.com/cpp-pm/hunter/archive/v0.23.251.tar.gz"
    SHA1 "5659b15dc0884d4b03dbd95710e6a1fa0fc3258d"
)

project(banking)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
option(BUILD_TESTS "Build tests" OFF)

hunter_add_package(GTest)
find_package(GTest CONFIG REQUIRED)

add_library(account STATIC banking/Account.cpp)
target_include_directories(account
 PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/banking)

add_library(transaction STATIC banking/Transaction.cpp)
target_include_directories(transaction
 PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/banking)
if(BUILD_TESTS)
  # Включить поддержку тестирования:
  enable_testing()
  # Создание списка файлов, соответствующих выражению и сохранение его в переменную
  file(GLOB ${PROJECT_NAME}_TEST_SOURCES tests/*.cpp)
  add_executable(check ${${PROJECT_NAME}_TEST_SOURCES})
  target_compile_options(check PRIVATE --coverage)
  target_link_libraries(check PRIVATE account transaction GTest::gtest_main GTest::gmock_main  --coverage)
  # Добавление тестов к проекту
  add_test(NAME check COMMAND check)
endif()
