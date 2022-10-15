---
title: C++20 Modules - 讓編譯加速吧 | C++ · 傳統與革新的空間
date: 2022-10-14 21:40:00
tags:
    - C++
    - C++20
    - C++20 Modules
categories:
    - C++ · 傳統與革新的空間
keywords:
    - C++
    - C++20
    - C++20 Modules
description:
cover: https://i.imgur.com/ojAq54F.jpg
---


# 前言

## Modules 的好處

以往，假如你在一個 cpp file 中 `#include` 了某個 header file，Preprocessor 會把你要的 header file 引入，變成一個 `translation unit` ，
但如果你有多個檔案都 `#include` 同一個 header，那 Preprocessor 就會每一個都引入一遍，造成**編譯速度緩慢**。

在引入 `C++20 Modules` 之後，編譯好的 modules 可以直接在各個地方被 compiler 利用，編譯速度就可以大幅提升。
當然，Modules 帶來好處不只編譯速度的提升，還有**封裝、引入順序不影響 macro** 等優點。

## Compiler 支持狀況

> [https://en.cppreference.com/w/cpp/compiler_support](https://en.cppreference.com/w/cpp/compiler_support)
**C++20 features > Modules**

至目前為止，只有 MSVC 的支持最完整，GCC , Clang 都只有 partial，因此我們這裡就以 MSVC 舉例。


# 配置 Visual Studio

## 在開始之前

在開始之前，我們需要設定一下 Visual Studio，

> [https://en.cppreference.com/w/cpp/compiler_support](https://en.cppreference.com/w/cpp/compiler_support)
**C++23 library features > Standard Library Modules**

由於 Standard Library Modules 在 C++23 才引入，
為了處理舊有的 header，我們可以使用 [`header units`](https://learn.microsoft.com/cpp/build/walkthrough-header-units?view=msvc-170)

( 其實 msvc 有預先實作，但 IntelliSense 等方面還有些問題，有興趣的人可以[自行嘗試](https://learn.microsoft.com/en-us/cpp/cpp/modules-cpp?view=msvc-170#consume-the-c-standard-library-as-modules) )

## Prerequisites

Visual Studio 2019 16.10 or later，這裡我直接用 Visual Studio 2022

## Project Properties

* **C++ Language Standard** 至少要設定 `/std:c++20`
**Configuration Properties > General > C++ Language Standard**

* **Scan Sources for Module Dependencies** 設定成 `Yes`
**Configuration Properties > C/C++ > General**

**副檔名**部分各家要求不太一樣，Visual Studio 要求的是 `Example.ixx`，
你可以直接在 solution explorer 中加入

![add module](https://i.imgur.com/4K2wMQS.png)


# 概述

這裡先舉個宣告跟實現合在一起的例子

```c++
// hello.ixx
export module helloworld; // module declaration

import <iostream>;        // import declaration

export void hello()       // export declaration
{
    std::cout << "Hello world!\n";
}
```

```c++
// main.cpp
import helloworld; // import declaration

int main()
{
    hello();
}
```

```
// Console
Hello world!

```

## hello.ixx

* `export module helloworld;`
宣告並匯出一個 **module** , 名字為 *helloworld* ，
加上 `export` 代表這是一個 **primary module interface unit**

* `import <iostream>; `
以 `header unit` 的方式 `import` 一個 header

* `export void hello();`
要匯出一個 function , 在宣告前加上 `export` 即可

## main.cpp

* `import helloworld;`
`import` 一個 module , 名字為 *helloworld*， module 的名字為前面 `export module` 的名稱，與檔名無關


# Module declarations - 宣告 Module

這裡我們稍微修改一下前一個例子

```c++
// hello.ixx
export module helloworld; // module declaration

import <iostream>;        // import declaration

export void hello();     // export declaration
```

```c++
// hello.cpp
module helloworld; // declares a module implementation unit for named module 'helloworld'

// export void hello() // ERROR: a declaration can be exported only from a module interface unit
void hello()       // implementation for 'void hello()'
{
    std::cout << "Hello world!\n";
}
```

```c++
// main.cpp
import helloworld; // import declaration

int main()
{
    hello();
}
```

Module 可以分為以下幾種 **module units**，
* module **interface** unit
* module **implementation** unit
* **primary** module interface unit
* module **partition interface** unit
* module **partition implementation** unit

## module interface unit

**module interface unit** ，指的是 `helloworld.ixx`，
在 **module declaration** 前**有加上** `export`，表示其為 `interface`，相似於之前的 `.h`，

負責宣告、導出 module name , namespaces , functions 等你想導出的東西，
module 的名字為 `export module module-name` 的 *module-name* 所定義，與檔名無關

## module implementation unit

**module implementation unit** ，指的是 `hello.cpp`，
在 **module declaration** 前沒有 `export`，表示其為 `implementation`，相似於之前的 `.cpp`，

負責**實現 interface file** 中的宣告，如同前面所說，名字為 *module-name* 所定義，與檔名無關。
> 注意，在 `implementation` 中也可以進行 `import`，但 `export` 只能在 `interface` 中使用。


## module partition units

**module partition interface unit , module partition implementation unit** ，
可以將單個 module 分成多個 `partition`，後面會再談到，基本性質跟前面大同小異

## primary module interface unit

**primary module interface unit** ，就這個例子來說，`helloworld.ixx` 符合其定義，
module 可能會因為 `partition` 的關係有多個 interface ，
但 **primary module interface** 每一個 module 都**只能也必須有一個**，
只要他不是宣告為 **module partition** ( `export module A:B;` )，那就會被視為 **primary module interface**，

## dot 句點

舉個例子來說，`mymodule.mysubmodule`
看到 **dot**，大家直覺應該都覺得是類似 class，
但在 module 中，dot 並**沒有特殊的含義**，就只是名稱的一部分，
然而，通常還是會把他拿來表示階層的關係


# Exporting declarations - 匯出宣告

`export` 除了可以用來匯出 module (`export module A;`) 外，也可以匯出**宣告**跟 **namespace**，
如果不想每個都打 `export` ，也可以把東西放在 `{ }` 裡，再全部 `export` 。

```c++
// hello.ixx
export module helloworld; // declares the primary module interface unit for named module 'helloworld'


// zero() will be visible by translations units importing 'helloworld'
export int zero() { return 0; }

// one() will NOT be visible.
int one()  { return 1; }

// Both two() and three() will be visible.
export
{
    int two()  { return 2; }
    int three() { return 3; }
}

// Exporting namespaces also works: number::four() and number::five() will be visible.
export namespace number
{
    int four()  { return 4; }
    int five() { return 5; }
}
```


# Importing modules and headers - 匯入 modules 跟 headers

## modules and headers

`import` 大致可以分為兩種，一種是一般的 `import module`，另一種是前面提到的 `import <headeer>`，
擺放的位置，要在 module declaration 後，在其他 declarations 前。
> 在 module 中，不應該直接使用 `#include`，如果要用，要把它放在 `global module fragment`

## export-import

透過 module，可以避免我們在 module units 中 `import` 的東西，被使用 module 的人連帶 `import`，
但如果你想，可以透過 `export` 將其再次匯出。

```c++
// A.ixx (primary module interface unit of 'A')
export module A;
 
import <iostream>; // import headeer
export import <string_view>; // export-imports
 
export void print(std::string_view message)
{
    std::cout << message << std::endl;
}
```

```c++
// main.cpp
import A; // import module

int main()
{
    std::string_view message = "Hello, world!";
    print(message);
}
```


# Global module fragment

如果因為 `macros` 的關係需要用到 `#define` 來設定 `headers` ，我們可以把它放在 `global module fragment` 裡。

位置在 module 的**最開始**，第一個宣告必須放 `module;`，範圍到 module declaration 結束。

```c++
// A.ixx (primary module interface unit of 'A')
module; // start
 
// https://learn.microsoft.com/en-us/cpp/c-runtime-library/math-constants?view=msvc-170
// Defining _USE_MATH_DEFINES to use Math Constants
#define _USE_MATH_DEFINES // for C++
#include <cmath>
 
export module A; // end

import <iostream>;

export void pi()
{
    std::cout << M_PI << std::endl; // 3.1415926
}
```


# Private module fragment

如果你想把宣告跟實現在單個檔案中完成，你也可以選擇把實現放在 `Private module fragment`。

位置在 module 的**最尾端**，範圍由 `module : private;` 開始。

```c++
export module A;
 
export int one();
 
module : private; // The start of the private module fragment.

int one()           
{
    return 1;
}
```


# Module partitions - 模組分區

最後這部分稍微有點複雜，
如同字面的意思，單個 Module 可以拆為多個 `partitions` (分區?)，
可以看成一個 **primary** module + 多個 **partition** module ，

partition module interface ，語法為**冒號後加名子**，`export module module-name:part-name`

* 命名習慣上，通常是 `<primary-module-name>-<module-partition-name>` , ( e.g, `A-B` , `A-C` )
* 一個 module partition 只能屬於一個 module，(`export module A:B;` 屬於 `A` )
* module partition 可以被其他 partition `import` , 語法為 `import :part-name`

在 **primary module interface unit**，除了 **implementation**，必須 `export` **所有 partition module interface** ，
語法為 `export import :part-name`，
以底下例子來說，就是
```c++
export import :B;    // partition module interface
export import :C;    // partition module interface
```
> 須 `export` 所有 **interface partitions** 的[規定](https://eel.is/c++draft/module.unit#3)是 [No diagnostic is required](https://en.cppreference.com/w/cpp/language/ndr)，
所以 compiler 不一定會提醒，請注意

```c++
// A.ixx
export module A;     // primary module interface unit

export import :B;    // Hello() is visible when importing 'A'.
export import :C;    // WorldImpl() is visible only for 'A.ixx'.

// World() is visible by any translation unit importing 'A'.
export void World()
{
    std::cout << WorldImpl() << '\n';
}

export int zero()
{
    return 0;
}
```

```c++
// A-B.ixx 
export module A:B; // partition module interface unit

// import :C;
import <iostream>;

// Hello() is visible by any translation unit importing 'A'.
export void Hello() 
{ 
	std::cout << "Hello" << '\n';
	// std::cout << WorldImpl() << '\n'; // ERROR: WorldImpl() is not visible.
}
```

```c++
// A-C.ixx
export module A:C; // partition module interface unit

// WorldImpl() is visible by any module unit of 'A' importing ':C'.
char const* WorldImpl() { return "World"; }
```

```c++
// main.cpp 
import A;

// import <iostream>;

int main()
{
    
    Hello();
    World();

    // std::cout << zero() << '\n'; // ERROR: 'cout': undeclared identifier
    // WorldImpl(); // ERROR: WorldImpl() is not visible.
}
```

比較特別的是，module partition 一樣可以用 **implementation unit** 的方式
但是要注意，implementation unit 是不能被 `export` 的，
> 另外到目前為止，在 MSVC 下，檔案的 **Compile as** 需設定成 `Module Internal Partition (/internalPartition )`
**C/C++ > Advanced > Compile as**

```c++
//  A.cpp   
export module A;     // primary module interface unit
 
import :C;           // WorldImpl() is now visible only for 'A.cpp'.
// export import :C; // ERROR: Cannot export a module implementation unit.
```

```c++
// A-C.cpp 
module A:C; // partition module **implementation** unit
 
// WorldImpl() is visible by any module unit of 'A' importing ':C'.
char const* WorldImpl() { return "World"; }
```


# 結語

Modules 的部分終於暫時寫完了，一開始看好像稍嫌複雜，其實不然，只是相同概念的組合
對於 C++20 的主要 feature ，算是大致介紹了一個，
用一篇就寫完好像還是太多了? 本來想多切成幾篇的說~


# References

* [Overview of modules in C++ | Microsoft Learn](https://learn.microsoft.com/en-us/cpp/cpp/modules-cpp?view=msvc-170)
* [Modules (since C++20) - cppreference.com](https://en.cppreference.com/w/cpp/language/modules)
* [C++20: Structure Modules - ModernesCpp.com](https://www.modernescpp.com/index.php/c-20-divide-modules)
* [visual c++ - C++ 20 experimental std Modules not accessible with latest MSVC experimental tools enabled - Stack Overflow](https://stackoverflow.com/questions/71866701/c-20-experimental-std-modules-not-accessible-with-latest-msvc-experimental-too)
* [Walkthrough: Build and import header units in Visual C++ projects | Microsoft Learn](https://learn.microsoft.com/en-us/cpp/build/walkthrough-header-units?view=msvc-170)
* [[C++20 Modules] Module Partition Implementation Unit Bug(?) - Visual Studio Feedback](https://developercommunity.visualstudio.com/t/c20-modules-module-partition-implementation-unit-b/1634347)
* [Understanding C++ Modules: Part 1: Hello Modules, and Module Units (vector-of-bool.github.io)](https://vector-of-bool.github.io/2019/03/10/modules-1.html)
* [[module.unit] (eel.is)](https://eel.is/c++draft/module.unit)
* [No Diagnostic Required - cppreference.com](https://en.cppreference.com/w/cpp/language/ndr)



---
Photo by [Oskar Kadaksoo](https://unsplash.com/@oskark?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/t/food-drink?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)

