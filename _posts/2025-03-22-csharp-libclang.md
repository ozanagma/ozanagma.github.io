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

<img src="{{site.baseurl | prepend: site.url}}/assets/images/clangsharp_ver.PNG" alt="ClangSharp Versiyon" />

Bu yüklemeyi yaptıktan sonra paketin içerisindeki:
1. ClangSharp.dll
2. ClangSharp.Interop.dll
3. libclang.dll
4. libClangSharp.dll kütüphaneleri exe dosyasının yanına kopyalanmalıdır.

CXIndex, Clang kütüphanesinde bir analiz "oturumunu" temsil eder. 
Bu nesne oluşturulmadan Clang ile hiçbir dosya analiz edilemez.
Clang motorunu kullanmak için gerekli olan altyapıyı hazırlar.
Bu bağlam üzerinden bir veya daha fazla dosya ayrıştırılabilir.
Çoğu durumda tek bir CXIndex yeterlidir. Ancak:
* Paralel olarak birden fazla analiz yapılacaksa,
* Ya da birbirinden bağımsız çeviri birimleri işlenecekse birden fazla CXIndex kullanılabilir.

```csharp
var index = CXIndex.Create();
```
 CXTranslationUnit.Parse işlevini bir cpp dosyası için çağırarak bu dosya için çeviri birimi(ing. translation unit) üretir.
 Bu birim kodda bir Soyut Sözdizimi Ağacı(ing. AST) olarak temsil edilir.
 
 Not: Çeviri birimi bir cpp veya hpp dosyasının ön işlemciden geçtikten sonraki haline denir. Derleyici bu birimi girdi olarak alır ve 
 çıktı olarak obje dosyası üretir.
```csharp
var tu = CXTranslationUnit.Parse(index, "filePath.hpp",
    commandLineArgs: Array.Empty<string>(),
    unsavedFiles: Array.Empty<CXUnsavedFile>(),
    options: CXTranslationUnit_Flags.CXTranslationUnit_None);
```

Çeviri birimi üretildikten sonra 
```csharp
CXCursorVisitor visitor = (CXCursor c, CXCursor parent, void* data) =>
{
    if (c.Kind == CXCursorKind.CXCursor_StructDecl)
    {
        Debug.WriteLine($"StructDecl: {c.Spelling}");

        var baseSpecCount = c.NumBases;
        for (uint i = 0; i < baseSpecCount; i++)
        {
            var baseType = c.GetBase(i);
            Debug.WriteLine($"BaseSpecifier: {baseType.Spelling}");
        }
    }

    if (c.Kind == CXCursorKind.CXCursor_FieldDecl)
    {
        var type = c.Type;
        Debug.WriteLine($"FieldDecl: {c.Spelling} (type: {type.Spelling})");
    }

    return CXChildVisitResult.CXChildVisit_Recurse;
};
```

```csharp
trUnit.Cursor.VisitChildren(visitor, (CXClientData)null);
```

```csharp
trUnit.Dispose();
```

```csharp
index.Dispose();
```