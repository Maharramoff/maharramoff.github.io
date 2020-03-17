---
layout: post
title: PHP 7-də yeni xüsusiyyətlər. 2) Return tip yoxlamaları və null birləşmə operatoru (??)
date: 2016-11-11 00:00
image: /assets/images/markdown.jpg
headerImage: false
projects: false
tag:
- php 7
category: blog
author: Maharramoff
description: PHP 7-də yeni xüsusiyyətlər. 2) Return tip yoxlamaları və null birləşmə operatoru (??)
---

### Return tip yoxlamaları

[Keçən mövzuda](/blog/2016/11/07/php-7-de-tip-yoxlamalari/) arqument tip yoxlamaları barədə yazdım. PHP 7-də həmçinin **return tip yoxlamaları** da mövcuddur. Return tipi cüt nöqtə ilə, parametrlər siyahısından dərhal sonra
fiqurlu mötərizədən əvvəl elan edilir. Bu xüsusiyyət də həmçinin zəif tip-yoxlama rejimindən asılı olaraq çalışır. Bildiyimiz kimi, `declare(strict_types=1);` sərt yoxlamanı aktiv edir, `declare(strict_types=0);` isə zəif yoxlamadır.

Belə bir numünə funksiyaya baxaq:
```php
function toDivide(int $a, int $b) : int
{
    return $a / $b;
}
```

Nümunədən anlaşılır ki, bölmə funksiyası iki **int** tipli parametr və həmçinin **int tipli return** gözləyir.

Hesab edək ki, zəif yoxlama rejimi aktivdir. Bu halda:
```php
var_dump(toDivide(20, 5)); 
// int(4)

var_dump(toDivide(20, 7)); 
// int(2)
```
Baxmayaraq ki, `toDivide(20, 7)` üçün aktual nəticə kəsr ədəd **(float)** olmalı idi, hazırki halda zəif yoxlama rejimi olduğu üçün return tipi konversiyanı işə salır və nəticəni int tipinə çevirir. İndi isə hesab edək ki, sərt yoxlama rejimi `declare(strict_types=1);` aktivdir. Bu halda nəticələr belə olacaq:
```php
var_dump(toDivide(20, 5)); 
// int(4)

var_dump(toDivide(20, 7)); 
Fatal error:  Uncaught TypeError: Return value of toDivide() must be of the type integer, float returned in ...
```
Daha bir nümunəyə iki halda baxaq:
```php
// Funksiya string tipi qaytarmalıdır
function viewString(string $s)
{ 
    echo $s;
}

declare(strict_types=0);
viewString(10); 
// "10"

declare(strict_types=1);
viewString(10); 
Fatal error: Uncaught TypeError: Argument 1 passed to viewString() must be of the type string, integer given, called in...
```

> ***Qeyd:** Return tipi elan olunmuş funksiya heç vaxt null qaytara bilməz və bu, tip\-yoxlama rejimindən asılı deyil.*

```php
function fooBar($n): bool 
{ 
    return $n > 0 ? true : null;
}
fooBar(0);
Uncaught TypeError: Return value of fooBar() must be of the type boolean, null returned in
```
### Null birləşmə operatoru (??)

C# proqramçılar ardını oxumaya da bilərlər. Mövzu onlara artıq başlıqdan aydın olmalı idi 😉

Bu xüsusiyyət çoxunuzu sevindirəcək. Artıq **$_POST**, **$_GET** və s.-lə gələn dəyərləri yoxlamaq əvvəlki kimi bezdirici olmayacaq! Əvvəlcə bir qədər keçmişə qayıdaq. PHP 5-də **ternary** adlanan operator bir-çox proqramçılara tanışdır. Bu operator aşağıdakı nümunəyə əsasən dəyəri yoxlayır, əgər yoxlama **true** olarsa ikinci elementi, **false** olarsa 3-cü elementi qaytarır.
```php
//=========1===========2============3======
$var = (10 > 5) ? 'Doğrudur!' : 'Səhvdir!'; 
// Doğrudur!

$var = (10 < 8) ? 'Doğrudur!' : 'Səhvdir!'; 
// Səhvdir!
```
WEB layihələri hazırlayarkən dəyişənlərin mövcudluğunu yoxlamaq demək olar ki, ən çox istifadə olunan əməliyyatlardandır. Bu əməliyyatlar əsasən bu formatda olur: Dəyişəni yoxlayırıq əgər yoxdursa hər hansı default bir dəyər veririk.
Məsələn, saytda dil paketi tətbiq olunub. Təxmini belə bir yoxlama etmək olardı:
```php
// Ən primitiv üsul
if(isset($_GET['lang']))
{ 
    $lang = $_GET['lang'];
}
else
{ 
    // Default dəyər
    $lang = 'az'; 
}
```
Ternar operator istifadə edənlər bilər ki, bu işi bir az da sadələşdirmək olurdu:
```php
// Daha qısa üsul
$lang = isset($_GET['lang']) ? $_GET['lang'] : 'az';
```
Null birləşmə operatoru **(??)** bunu daha yığcam yazmağa imkan verir!
```php
// Artıq isset yazmağa gərək yoxdur!
$lang = $_GET['lang'] ?? 'az'; 
```
Yuxarıdakı nümunədə **??** operatoru ilk olaraq **$_GET['lang']** dəyişəninə avtomatik olaraq **isset** yoxlamasını tətbiq edir, əgər dəyişənin dəyəri **null** deyilsə dərhal onu, əks halda **az** dəyərini qaytarır.

**Chaining (Zəncirvari)**    
Bundan başqa, hətta daha geniş yoxlamalarda bu operatordan bir-neçə dəfə istifadə edə bilərik. Soldan sağa gedərək ilk tapılan mövcud olan və null olmayan dəyər qaytarılır.

```php
// Nümunə 1
$a = null; // və ya ümumiyyətlə belə dəyişən təyin edilməyib
$b = 10;
echo $a ?? 30; // 30
echo $a ?? $b ?? 40; // 10

// Nümunə 2
$a = ["key3" => "abc"];
var_dump($a["key1"] ?? $a["key2"] ?? $a["key3"]); 
// string(3) "abc"

// Nümunə 3 + Ternar operator
var_dump(1 ?? 3 ? 5 : 7); 
// int(5)

// Nümunə 4 daha mürəkkəb görünür
var_dump(false || null ?? false ? false : true); // bool(true)
```
