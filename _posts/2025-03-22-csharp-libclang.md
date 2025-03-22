---
layout: post
title: "C#'da libclang kullanımı"
date: 2025-03-22
categories: [yazılım, csharp]
tags: [libclang, C#, parser, clang, interop]
---

C# dilinde `libclang` kullanarak C++ kodlarını ayrıştırmak mümkün mü?  
Bu yazıda `libclang` kütüphanesini C# ile nasıl kullanabileceğimizi adım adım göstereceğim.

Hadi başlayalım! 🚀

Olabildiğince çok tipin olduğu yapabildiğimiz kadar karmaşık bir struct yapalım. 
Aşağıdaki struct hem başka bir structtan türüyor ve türediği struct'ın bazı elemanlarını ilkliyor.
hem içerisinde bir enum tutuyor, hem de yine bir eleman olarak bir namespace içerisinde struct barındırıyor.


```c++
#pragma once

#include <cstdint>

#pragma pack(push, 1)

struct STestBase
{
  uint16_t Base0;
  uint16_t Base1;
};

enum class ETest
{
  Test0,
  Test1,
  Test2,
};

namespace NTest
{
  struct STest
  {
    float Test1;
    double Test2;
  };
}

struct STest : STestBase
{
  uint64_t Test0;  
  uint32_t Test1;
  uint16_t Test2;
  uint8_t Test3;

  int64_t Test4;
  int32_t Test5;
  int16_t Test6;
  int8_t Test7;

  float Test8;
  double Test9;

  uint8_t Test10[10];
	
  bool Test11;
  char Test12;

  NTest::STest Test13;
};

#pragma pack(pop)
```

Bu dosyayı cpp elemanlarına ayrıştırmak için Tools → Nuget Package Manager → Manage Nuget Packages for Solutions 'a gidip
ClangSharp paketini yüklemeliyiz. Bu yazının yazıldığı tarihte en güncel ve kararlı sürüm 18.1.0.3'tür.

![ClangSharp Versiyon](/assets/images/clangsharp_ver.png)
