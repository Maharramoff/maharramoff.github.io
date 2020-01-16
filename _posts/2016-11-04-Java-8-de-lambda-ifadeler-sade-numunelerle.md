---
layout: post
title: Java 8-də Lambda ifadələr. Sadə nümunələrlə.
date: 2016-11-04 00:00
image: /assets/images/posts/java-8-lambdas.jpg
headerImage: true
tag:
- java
- lambda
category: blog
author: Maharramoff
description: Java 8-də Lambda ifadələr. Sadə nümunələrlə.
---

Java 8-in ən maraqla gözlənilən xüsusiyyətlərindən biri Lambda ifadələr (Anonim funksiya/metod) oldu. Lambdalar bir sətirlə kiçik kod yazmaq imkanı yaratmaqla Javada funksional üslub proqramlaşdırmanı daha da sadələşdirir.



**Lambda ifadələrin strukturu**

*   Lambda ifadələr adətən belə sintaksislə `(argument) -> (body)` yazılır  
*   Lambda ifadələrdə sıfır, bir və ya bir-neçə parametr ola bilər
*   Parametr tipləri həm aşkar elan edilə bilər, həm də mövcud kontekstdən çıxarıla bilər. `(int a)` və ya sadəcə `(a)` kimi verilə bilər.
*   Parametrlər mötərizə daxilində olur və vergüllə ayrılır. Məsələn, `(a, b)` və ya `(String a, double b, int c)`
*   Boş (sıfır) parametr verərkən sadəcə boş mötərizə açılır. Məsələn, `() -> 35`
*   Sadəcə bir parametr ötürülürsə və onun qaytaracağı tip məlumdursa mötərizə yazmaq vacib deyil. Məsələn, `a -> return a+a`
*   Əgər lambda ifadənin daxilində birdən çox şərt/operator/statement olarsa o halda onlar fiqurlu mötərizəyə (kod bloku) alınmalıdır. `(a) -> {statement1; statement2; ...;};`

**Lambda ifadələrə nümunələr**

Normalda verilmiş qovluqdan bütün .jpg sonluqlu faylları (anonim class istifadə edərək) belə göstərə bilərik:
```java
// Köhnə üsul
File dir = new File("/images");

String[] imglist = dir.list(new FilenameFilter()
{
    public boolean accept(File f, String s)
    {
        return s.endsWith(".jpg");
    }
});
```
Lambda ifadəylə bu daha da sadələşdirilə bilər
```java
// Yeni üsul
File dir = new File("/images");
String[] imglist = dir.list((f, s) -> { return s.endsWith(".jpg"); });
```
Daha bir sadə nümunədə müxtəlif üsullarla massivin bütün elementlərini çap edirik.

```java
// Verilmiş massiv
List<String> list = Arrays.asList("a", "b", "c", "d", "e", "f");

// Köhnə üsul:
for(String s: list)
{
    System.out.print(s); // abcdef
}

// Yeni üsul:
list.forEach(s -> System.out.print(s)); // abcdef

// Və ya tipi aşkar göstərməklə
list.forEach((String s) -> System.out.print(s)); // abcdef

// Və ya daha sadələşmiş formada cüt iki nöqtə operatorundan :: istifadə etmək olar
list.forEach(System.out::print); // abcdef
```
**Funksional proqramlaşdırma və Lambdalar**

Tutaq ki, hər hansı massiv verilib və bizə bu massivin elementlərini cəmləmək tapşırılıb.

```java
List<Integer> list = Arrays.asList(5, 6, 7, 9, 11, 12, 14, 15, 16, 18, 20, 25, 30);
```

Normalda bunu belə həll edərik
```java
public static int arraySum(List<Integer> elements)
{
    int total = 0;
    for (int n : elements)
    {
        total += n;
    }
    
    return total;
}

System.out.print(arraySum(list)); // 188
```
Sonra isə bu massivə aid başqa bir tapşırıq gəlir. Bu dəfə massivin sadəcə tək ədədlərini cəmləməyi tapşırırlar.

```java
public static int arraySumOdds(List<Integer> elements)
{
    int total = 0;
    for (int n : elements)
    {
        total += (n & 1) != 0 ? n : 0;
    } 
    
	return total;
}

System.out.print(arraySumOdds(list)); // 72
```
Bu şəkildə tapşırıqlar artdıqca hər dəfə yeni metod yaza-yaza gedərik. Amma Lambda ifadənin və funksional interfeysin köməyi ilə bu işi çox sadələşdirmək olar.

> ***Qeyd 1:** Funksional interfeys mütləq şəkildə 1 abstrakt metoddan ibarət olmalıdır.*

**1)** Massivdə yoxlama işlərini yerinə yetirəcək `VerifyArray` adlı bir interfeys və arqument yoxlamadan keçdikdə `true` əks halda `false` qaytaran abstrakt bir metod `verify` elan edirik.
```java
interface VerifyArray<T>
{
    boolean verify(T t);
}
```

> ***Qeyd 2:** Əslində yazdığımız interfeysin analoqu default olaraq artıq Java 8-də **java.util.function** paketi ilə əlavə olunub. Predicate adlanır. Sadəcə nümunə aydın olsun deyə interfeysi tək metodla özümüz yaratdıq.*

**2)** Sadəcə tək bir funksiya yazırıq. Funksiyada toplama əməliyyatından qabaq **VerifyArray** interfeysi ədədləri hansı tip yoxlamadan keçirəcəyimizi təyin edir.
```java
public static int arraySumAll(List<Integer> list, VerifyArray<Integer> value)
{
    int total = 0;
    for(Integer n: list)
    {
        // Yoxlama
        if(value.verify(n)) 
        {
            total += n;
        }
    }

    return total;
}
```
**3)** İndi isə bayaq etdiyimiz və əlavə bir neçə əməliyyatı, yazdığımız funksiya və lambda ifadələrin köməyi ilə həll edək

```java
// Bütün ədədləri topla
arraySumAll(list, (n)-> true) // 188

// Heç bir ədədi toplama
arraySumAll(list, (n)-> false) // 0

// Ancaq cüt ədədləri topla
arraySumAll(list, (n)-> (n & 1) == 0) // 116

// Ancaq tək ədədləri topla
arraySumAll(list, (n)-> (n & 1) != 0) // 72

// Ancaq 9-dan böyük ədədləri topla
arraySumAll(list, (n)-> n > 9) // 161

// Ancaq 3-ə bölünən və 6-dan böyük ədədləri topla
arraySumAll(list, (n)-> n > 6 && n % 3 == 0) // 84

// və s. və s...
```
Bunlar sadəcə tanışlıq üçün sadə nümunələr idi. Əslində Lambdalar daha geniş bir mövzudur və böyük imkanları var.
