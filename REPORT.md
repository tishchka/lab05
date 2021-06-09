## Laboratory work V

Данная лабораторная работа посвещена изучению фреймворков для тестирования на примере **GTest**

```sh
$ open https://github.com/google/googletest
```

## Tasks

- [x] 1. Создать публичный репозиторий с названием **lab05** на сервисе **GitHub**
- [x] 2. Выполнить инструкцию учебного материала
- [x] 3. Ознакомиться со ссылками учебного материала
- [x] 4. Составить отчет и отправить ссылку личным сообщением в **Slack**

## Tutorial

```sh
# Создаем перменную с именем пользователя на GitHub
$ export GITHUB_USERNAME=tishchka
# Заменяем команду gsed на команду sed
$ alias gsed=sed # for *-nix system
```

```sh
# Переходим в директорию workspace
$ cd ${GITHUB_USERNAME}/workspace
# Запоминаем текущий каталог в виртуальном стеке каталогов
$ pushd .
~/tishchka/workspace ~/GNDavydov/workspace

# Исполняем команду из файла scripts/activate
$ source scripts/activate
```

```sh
# Клонируем репозиторий, созданный во 4-ой лабораторной,
# в папку с лабораторной №5
$ git clone https://github.com/${GITHUB_USERNAME}/lab04 projects/lab05
Cloning into 'projects/lab05'...
remote: Enumerating objects: 26, done.
remote: Counting objects: 100% (26/26), done.
remote: Compressing objects: 100% (17/17), done.
remote: Total 26 (delta 3), reused 26 (delta 3), pack-reused 0
Unpacking objects: 100% (26/26), 4.21 KiB | 861.00 KiB/s, done.

# Переходим в созданную директорию
$ cd projects/lab05
# Удаляем ссылку на репозиторий 4-ой лабораторной
$ git remote remove origin
# Добывляем ссылку с именем origin на созданный репозиторий 5-ой лабораторной
$ git remote add origin https://github.com/${GITHUB_USERNAME}/lab05
```

```sh
# Создаем директорию third-party
$ mkdir third-party
# Создаем подмодуль, куда помещаем библиотеку gtest,
# склонируемую с ссылки
$ git submodule add https://github.com/google/googletest third-party/gtest
Cloning into '/home/tishchka/tishchka/workspace/projects/lab05/third-party/gtest'...
remote: Enumerating objects: 8, done.
remote: Counting objects: 100% (8/8), done.
remote: Compressing objects: 100% (8/8), done.
remote: Total 22324 (delta 1), reused 2 (delta 0), pack-reused 22316
Receiving objects: 100% (22324/22324), 8.57 MiB | 2.11 MiB/s, done.
Resolving deltas: 100% (16504/16504), done.

# Переходим в созданную директорию, создаем ветку release-1.8.1
# и возвращаемся обратно в рабочую директорию
$ cd third-party/gtest && git checkout release-1.8.1 && cd ../..
Note: switching to 'release-1.8.1'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by switching back to a branch.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -c with the switch command. Example:

  git switch -c <new-branch-name>

Or undo this operation with:

  git switch -

Turn off this advice by setting config variable advice.detachedHead to false

HEAD is now at 2fe3bd99 Merge pull request #1433 from dsacre/fix-clang-warnings

# Добавляем директорию gtest для отслеживания
$ git add third-party/gtest
# Сохраняем изменения
$ git commit -m"added gtest framework"
[main 65fc3b4] added gtest framework
 2 files changed, 4 insertions(+)
 create mode 100644 .gitmodules
 create mode 160000 third-party/gtest
```

```sh
# Добавляем в CMakeLists.txt строчку option(BUILD_TESTS "Build tests" OFF) 
# после option(BUILD_EXAMPLES "Build examples" OFF)
# В которой определяем переменную, которая ответственна за постройку тестов
$ gsed -i '/option(BUILD_EXAMPLES "Build examples" OFF)/a\
option(BUILD_TESTS "Build tests" OFF)
' CMakeLists.txt

# Добавляем в CMakeLists.txt следующий блок, который ответственный за сборку тестов
$ cat >> CMakeLists.txt <<EOF
if(BUILD_TESTS)
  enable_testing()
  add_subdirectory(third-party/gtest)
  file(GLOB \${PROJECT_NAME}_TEST_SOURCES tests/*.cpp)
  add_executable(check \${\${PROJECT_NAME}_TEST_SOURCES})
  target_link_libraries(check \${PROJECT_NAME} gtest_main)
  add_test(NAME check COMMAND check)
endif()
EOF
```

```sh
# Создаем директорию tests
$ mkdir tests

# Создаем файл test1.cpp и записываем туда тест,
# проверяющий работу функции print
$ cat > tests/test1.cpp <<EOF
#include <print.hpp>

#include <gtest/gtest.h>

TEST(Print, InFileStream)
{
  std::string filepath = "file.txt";
  std::string text = "hello";
  std::ofstream out{filepath};

  print(text, out);
  out.close();

  std::string result;
  std::ifstream in{filepath};
  in >> result;

  EXPECT_EQ(result, text);
}
EOF
```

```sh
# Генерируем и настраиваем наш проект, изменив значение
# переменной BUILD_TESTS на true
$ cmake -H. -B_build -DBUILD_TESTS=ON
-- The C compiler identification is GNU 9.3.0
-- The CXX compiler identification is GNU 9.3.0
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Found PythonInterp: /usr/bin/python3.8 (found version "3.8.5") 
-- Looking for pthread.h
-- Looking for pthread.h - found
-- Performing Test CMAKE_HAVE_LIBC_PTHREAD
-- Performing Test CMAKE_HAVE_LIBC_PTHREAD - Failed
-- Check if compiler accepts -pthread
-- Check if compiler accepts -pthread - yes
-- Found Threads: TRUE  
-- Configuring done
-- Generating done
-- Build files have been written to: /home/tishchka/tishchka/workspace/projects/lab05/_build

# Собираем наш проект
$ cmake --build _build
Scanning dependencies of target print
[  8%] Building CXX object CMakeFiles/print.dir/sources/print.cpp.o
[ 16%] Linking CXX static library libprint.a
[ 16%] Built target print
Scanning dependencies of target gtest
[ 25%] Building CXX object third-party/gtest/googlemock/gtest/CMakeFiles/gtest.dir/src/gtest-all.cc.o
[ 33%] Linking CXX static library libgtest.a
[ 33%] Built target gtest
Scanning dependencies of target gtest_main
[ 41%] Building CXX object third-party/gtest/googlemock/gtest/CMakeFiles/gtest_main.dir/src/gtest_main.cc.o
[ 50%] Linking CXX static library libgtest_main.a
[ 50%] Built target gtest_main
Scanning dependencies of target check
[ 58%] Building CXX object CMakeFiles/check.dir/tests/test1.cpp.o
[ 66%] Linking CXX executable check
[ 66%] Built target check
Scanning dependencies of target gmock
[ 75%] Building CXX object third-party/gtest/googlemock/CMakeFiles/gmock.dir/src/gmock-all.cc.o
[ 83%] Linking CXX static library libgmock.a
[ 83%] Built target gmock
Scanning dependencies of target gmock_main
[ 91%] Building CXX object third-party/gtest/googlemock/CMakeFiles/gmock_main.dir/src/gmock_main.cc.o
[100%] Linking CXX static library libgmock_main.a
[100%] Built target gmock_main

# Собираем библиотеку test
$ cmake --build _build --target test
Running tests...
Test project /home/tishchka/tishchka/workspace/projects/lab05/_build
    Start 1: check
1/1 Test #1: check ............................   Passed    0.00 sec

100% tests passed, 0 tests failed out of 1

Total Test time (real) =   0.00 sec
```

```sh
# Запускаем все тесты проекта
$ _build/check
Running main() from /home/tishchka/tishchka/workspace/projects/lab05/third-party/gtest/googletest/src/gtest_main.cc
[==========] Running 1 test from 1 test case.
[----------] Global test environment set-up.
[----------] 1 test from Print
[ RUN      ] Print.InFileStream
[       OK ] Print.InFileStream (0 ms)
[----------] 1 test from Print (0 ms total)

[----------] Global test environment tear-down
[==========] 1 test from 1 test case ran. (1 ms total)
[  PASSED  ] 1 test.

$ cmake --build _build --target test -- ARGS=--verbose
Running tests...
UpdateCTestConfiguration  from :/home/tishchka/tishchka/workspace/projects/lab05/_build/DartConfiguration.tcl
UpdateCTestConfiguration  from :/home/tishchka/tishchka/workspace/projects/lab05/_build/DartConfiguration.tcl
Test project /home/tishchka/tishchka/workspace/projects/lab05/_build
Constructing a list of tests
Done constructing a list of tests
Updating test list for fixtures
Added 0 tests to meet fixture requirements
Checking test dependency graph...
Checking test dependency graph end
test 1
    Start 1: check

1: Test command: /home/tishchka/tishchka/workspace/projects/lab05/_build/check
1: Test timeout computed to be: 10000000
1: Running main() from /home/tishchka/tishchka/workspace/projects/lab05/third-party/gtest/googletest/src/gtest_main.cc
1: [==========] Running 1 test from 1 test case.
1: [----------] Global test environment set-up.
1: [----------] 1 test from Print
1: [ RUN      ] Print.InFileStream
1: [       OK ] Print.InFileStream (0 ms)
1: [----------] 1 test from Print (0 ms total)
1: 
1: [----------] Global test environment tear-down
1: [==========] 1 test from 1 test case ran. (1 ms total)
1: [  PASSED  ] 1 test.
1/1 Test #1: check ............................   Passed    0.00 sec

100% tests passed, 0 tests failed out of 1

Total Test time (real) =   0.00 sec
```

```sh
# Изменяем значение для иконки travis для лабораторной 5
$ gsed -i 's/lab04/lab05/g' README.md
# Добавляем строку со значением переменной BUILD_TESTS
$ gsed -i 's/\(DCMAKE_INSTALL_PREFIX=_install\)/\1 -DBUILD_TESTS=ON/' .travis.yml
# Добавляем строку с командой
$ gsed -i '/cmake --build _build --target install/a\
- cmake --build _build --target test -- ARGS=--verbose
' .travis.yml
```

```sh
# Проверяем на отсутствие предупрежденй для .travis.yml
$ travis lint
Hooray, .travis.yml looks valid :)
```

```sh
# Добавляем файлы для отслеживания
$ git add .travis.yml
$ git add tests
# Смотрим изменения по файлам и одобряем их
$ git add -p
diff --git a/CMakeLists.txt b/CMakeLists.txt
index 96a361e..aa7a323 100644
--- a/CMakeLists.txt
+++ b/CMakeLists.txt
@@ -4,6 +4,7 @@ set(CMAKE_CXX_STANDARD 11)
 set(CMAKE_CXX_STANDARD_REQUIRED ON)
 
 option(BUILD_EXAMPLES "Build examples" OFF)
+option(BUILD_TESTS "Build tests" OFF)
 
 project(print)
 
(1/2) Stage this hunk [y,n,q,a,d,j,J,g,/,e,?]? y
@@ -34,3 +35,12 @@ install(TARGETS print
 
 install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/ DESTINATION include)
 install(EXPORT print-config DESTINATION cmake)
+
+if(BUILD_TESTS)
+  enable_testing()
+  add_subdirectory(third-party/gtest)
+  file(GLOB ${PROJECT_NAME}_TEST_SOURCES tests/*.cpp)
+  add_executable(check ${${PROJECT_NAME}_TEST_SOURCES})
+  target_link_libraries(check ${PROJECT_NAME} gtest_main)
+  add_test(NAME check COMMAND check)
+endif()
(2/2) Stage this hunk [y,n,q,a,d,K,g,/,e,?]? y

diff --git a/README.md b/README.md
index 83fbab2..0f883d5 100644
--- a/README.md
+++ b/README.md
@@ -1,2 +1,2 @@
-[![Build Status](https://travis-ci.org/tishchka/lab04.svg?branch=main)](https://travis-ci.org/tishchka/lab04)
+[![Build Status](https://travis-ci.org/tishchka/lab05.svg?branch=main)](https://travis-ci.org/tishchka/lab05)
 This is an example Print program in c++.
(1/1) Stage this hunk [y,n,q,a,d,e,?]? y

# Сохраняем изменения
$ git commit -m"added tests"
[main 06726fe] added tests
 4 files changed, 32 insertions(+), 2 deletions(-)
 create mode 100644 tests/test1.cpp

# Отправляем изменения на сервер
$ git push origin main
Username for 'https://github.com': tishchka
Password for 'https://tishchka@github.com': 
Enumerating objects: 37, done.
Counting objects: 100% (37/37), done.
Delta compression using up to 2 threads
Compressing objects: 100% (29/29), done.
Writing objects: 100% (37/37), 5.34 KiB | 237.00 KiB/s, done.
Total 37 (delta 8), reused 0 (delta 0)
remote: Resolving deltas: 100% (8/8), done.
To https://github.com/tishchka/lab05
 + 006b8ba...06726fe main -> main (forced update)
```

```sh
# Авторизируемся
$ travis login --auto
Successfully logged in as tishchka!

# Проверяем на доступность 
$ travis enable
Detected repository as tishchka/lab05, is this correct? |yes| yes
GNDavydov/lab05: enabled :)
```

```sh
# Создаем директорию artifacts
$ mkdir artifacts
# Делаем скриншот экрана
$ sleep 20s && gnome-screenshot --file artifacts/screenshot.png
# for macOS: $ screencapture -T 20 artifacts/screenshot.png
# open https://github.com/${GITHUB_USERNAME}/lab05
```

## Report

```sh
$ popd
$ export LAB_NUMBER=05
$ git clone https://github.com/tp-labs/lab${LAB_NUMBER} tasks/lab${LAB_NUMBER}
$ mkdir reports/lab${LAB_NUMBER}
$ cp tasks/lab${LAB_NUMBER}/README.md reports/lab${LAB_NUMBER}/REPORT.md
$ cd reports/lab${LAB_NUMBER}
$ edit REPORT.md
$ gist REPORT.md
```

## Homework

### Задание
1. Создайте `CMakeList.txt` для библиотеки *banking*.
2. Создайте модульные тесты на классы `Transaction` и `Account`.
    * Используйте mock-объекты.
    * Покрытие кода должно составлять 100%.
3. Настройте сборочную процедуру на **TravisCI**.
4. Настройте [Coveralls.io](https://coveralls.io/).

## Links

- [C++ CI: Travis, CMake, GTest, Coveralls & Appveyor](http://david-grs.github.io/cpp-clang-travis-cmake-gtest-coveralls-appveyor/)
- [Boost.Tests](http://www.boost.org/doc/libs/1_63_0/libs/test/doc/html/)
- [Catch](https://github.com/catchorg/Catch2)

```
Copyright (c) 2015-2021 The ISC Authors
```
