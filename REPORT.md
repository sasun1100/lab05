# Отчёт по лабораторной работе №5

**Студент:** Maksim Alehanov
**Группа:** ИУ8-25
**Тема:** Изучение фреймворков для тестирования на примере GTest

---

## 1. Цель работы

Научиться писать модульные тесты для C++ кода с помощью Google Test и
запускать их в CI. Тестируется библиотека `print`.

## 2. Выполнение

### 2.1. Подключение Google Test

GTest добавлен как git submodule:

```sh
mkdir third-party
git submodule add https://github.com/google/googletest third-party/gtest
cd third-party/gtest && git checkout v1.14.0 && cd ../..
```

### 2.2. Тесты

В директории `tests/` создан файл `test_print.cpp`:

```cpp
#include <sstream>
#include <gtest/gtest.h>
#include <print.hpp>

TEST(PrintTest, WritesPlainString) {
  std::ostringstream os;
  print("hello", os);
  EXPECT_EQ(os.str(), "hello");
}

TEST(PrintTest, WritesEmptyString) {
  std::ostringstream os;
  print("", os);
  EXPECT_EQ(os.str(), "");
}

TEST(PrintTest, AppendsConsecutiveWrites) {
  std::ostringstream os;
  print("foo", os);
  print("bar", os);
  EXPECT_EQ(os.str(), "foobar");
}

TEST(PrintTest, KeepsSpecialCharacters) {
  std::ostringstream os;
  print("a\nb\tc", os);
  EXPECT_EQ(os.str(), "a\nb\tc");
}
```

### 2.3. Настройка CMake

В `CMakeLists.txt` добавлена опция `BUILD_TESTS`:

```cmake
if(BUILD_TESTS)
  enable_testing()
  set(INSTALL_GTEST OFF CACHE BOOL "" FORCE)
  add_subdirectory(third-party/gtest)

  add_executable(check tests/test_print.cpp)
  target_link_libraries(check print gtest_main)

  add_test(NAME check COMMAND check)
endif()
```

### 2.4. Запуск тестов

```sh
$ cmake -H. -B_build -DBUILD_TESTS=ON
$ cmake --build _build
$ ctest --test-dir _build --output-on-failure
Test project _build
    Start 1: check
1/1 Test #1: check ............................   Passed    0.14 sec
100% tests passed, 0 tests failed out of 1
```

### 2.5. CI

Workflow GitHub Actions обновлён: тесты собираются и запускаются на gcc и
clang при каждом push (`cmake -DBUILD_TESTS=ON` + `ctest`).

## 3. Результаты

- Подключён GTest, написаны модульные тесты для библиотеки `print`.
- Все тесты проходят (4 теста).
- Тесты автоматически запускаются в CI на двух компиляторах.
