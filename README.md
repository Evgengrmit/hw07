[![Build Status](https://travis-ci.com/Evgengrmit/hw07.svg?branch=master)](https://travis-ci.com/Evgengrmit/hw07)
[![Build status](https://ci.appveyor.com/api/projects/status/4e9t9x09lm60mb0t/branch/master?svg=true)](https://ci.appveyor.com/project/Evgengrmit/hw07/branch/master)
[![Coverage Status](https://coveralls.io/repos/github/Evgengrmit/hw07/badge.svg?branch=master)](https://coveralls.io/github/Evgengrmit/hw07?branch=master)
## Homework VII

### Задание
Создадим настройки менеджера пакетов **Hunter** для репозитория с ДЗ № 5
Настройка git-репозитория **hw07** для работы
```sh
% git clone https://github.com/${GITHUB_USERNAME}/hw05 hw07
% cd hw07
% git remote remove origin
% git remote add origin https://github.com/${GITHUB_USERNAME}/lab07
```
Скачивание и подключение модуля `HunterGate`
```sh
% mkdir -p cmake # Создание директории где будут храниться файлы Hunter
# Скачивание данных из файла в удаленном репозитории и их запись в файл HunterGate.cmake
% wget https://raw.githubusercontent.com/cpp-pm/gate/master/cmake/HunterGate.cmake -O cmake/HunterGate.cmake
--2020-05-06 17:48:12--  https://raw.githubusercontent.com/cpp-pm/gate/master/cmake/HunterGate.cmake
Распознаётся raw.githubusercontent.com (raw.githubusercontent.com)… 151.101.112.133
Подключение к raw.githubusercontent.com (raw.githubusercontent.com)|151.101.112.133|:443... соединение установлено.
HTTP-запрос отправлен. Ожидание ответа… 200 OK
Длина: 17070 (17K) [text/plain]
Сохранение в: «cmake/HunterGate.cmake»

cmake/HunterGate.cmake   100%[===============================>]  16,67K  --.-KB/s    за 0,04s   

2020-05-06 17:48:12 (436 KB/s) - «cmake/HunterGate.cmake» сохранён [17070/17070]


# Добавление HunterGate к CMake
% gsed -i "" '/cmake_minimum_required(VERSION 3.10)/a\

include("cmake/HunterGate.cmake")
HunterGate(
    URL "https:\//github.com/cpp-pm/hunter/archive/v0.23.251.tar.gz"
    SHA1 "5659b15dc0884d4b03dbd95710e6a1fa0fc3258d"
)
' CMakeLists.txt
```
Теперь не нужно скачивать **GTest** самостоятельно. **Hunter** сам подтянет добавленные с помощью функции `hunter_add_package`.
```sh
# Удаление подмодуля с GTest
% git rm -rf third-party/gtest
rm 'third-party/gtest'
# Добавление через hunter пакета gtest и его поиск
% gsed -i "" '/option(BUILD_TESTS "Build tests" OFF)/a\

hunter_add_package(GTest)
find_package(GTest CONFIG REQUIRED)
' CMakeLists.txt
# Удаление строки с добавлением поддиректории gtest
% gsed -i "" 's/add_subdirectory(third-party/gtest)//' CMakeLists.txt
# Замена обращение к gtest gtest_main на GTest::gtest_main
% gsed -i "" 's/gtest_main/GTest::gtest_main/' CMakeLists.txt
```
Сборка прокта при помощи **Hunter**.
```sh
# Видим как полключаются пакеты при помощи Hanter'a
% cmake -H. -B_builds -DBUILD_TESTS=ON
...
-- Build files have been written to: /Users/evgengrmit/Evgengrmit/workspace/projects/hw07/_builds
% cmake --build _builds
Scanning dependencies of target account
[ 14%] Building CXX object CMakeFiles/account.dir/banking/Account.cpp.o
[ 28%] Linking CXX static library libaccount.a
[ 28%] Built target account
Scanning dependencies of target transaction
[ 42%] Building CXX object CMakeFiles/transaction.dir/banking/Transaction.cpp.o
[ 57%] Linking CXX static library libtransaction.a
[ 57%] Built target transaction
Scanning dependencies of target check
[ 71%] Building CXX object CMakeFiles/check.dir/tests/test1.cpp.o
[ 85%] Building CXX object CMakeFiles/check.dir/tests/test2.cpp.o
[100%] Linking CXX executable check
[100%] Built target check
% cmake --build _builds --target test
Running tests...
Test project /Users/evgengrmit/Evgengrmit/workspace/projects/hw07/_builds
    Start 1: check
1/1 Test #1: check ............................   Passed    0.01 sec

100% tests passed, 0 tests failed out of 1

Total Test time (real) =   0.01 sec# Вывод файлов из директории .hunter
% ls -la $HOME/.hunter
total 0
drwxr-xr-x   3 evgengrmit  staff    96  1 апр 19:55 .
drwxr-xr-x+ 35 evgengrmit  staff  1120  6 май 15:23 ..
drwxr-xr-x   7 evgengrmit  staff   224  6 май 16:18 _Base
```
Добавление конфигурационного файла в проект, который будет содержать необходимую версию GTest.
```sh
% mkdir cmake/Hunter
# Установка нужной версии GTest
% cat > cmake/Hunter/config.cmake <<EOF
hunter_config(GTest VERSION 1.7.0-hunter-9)
EOF
# add LOCAL in HunterGate function
```
Добавление подмодуля **polly**, который содержит инструкции для сборки проектов с установленным **Hunter**.
```sh
% mkdir tools
% git submodule add https://github.com/ruslo/polly tools/polly
% tools/polly/bin/polly.py --test
...
Generate: 0:00:03.640991s
Build: 0:00:01.585695s
Test: 0:00:00.013631s
-
Total: 0:00:05.240688s
-
SUCCESS
# Сборка проекта на компиляторе со станартом с++14
% tools/polly/bin/polly.py --toolchain clang-cxx14
...
Generate: 0:00:03.759353s
Build: 0:00:01.613048s
-
Total: 0:00:05.372811s
-
SUCCESS
```
Добавим непрерывную интеграцию с **Appveyor**
Создание `appveyor.yml`
```sh
% cat >> appveyor.yml <<EOF
image: Visual Studio 2019
platform:
  - x86
  - x64
configuration: Release

build_script:
  - cmd: cmake -H. -B_build -DBUILD_TESTS=ON
  - cmd: cmake --build _build
  - cmd: cmake --build _build --target test
  - cmd: _build/check
  - cmd: cmake --build _build --target test -- ARGS=--verbose
EOF
```
## Links

- [Create Hunter package](https://docs.hunter.sh/en/latest/creating-new/create.html)
- [Custom Hunter config](https://github.com/ruslo/hunter/wiki/example.custom.config.id)
- [Polly](https://github.com/ruslo/polly)

```
Copyright (c) 2015-2020 The ISC Authors
```
