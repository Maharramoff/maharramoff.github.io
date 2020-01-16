---
layout: post
title: PHP 7-də yeni xüsusiyyətlər. 1) Tip yoxlamaları.
date: 2016-11-07 00:00
image: /assets/images/markdown.jpg
headerImage: false
tag:
- php 7
category: blog
author: Maharramoff
description: PHP 7-də yeni xüsusiyyətlər. 1) Tip yoxlamaları.
---

Tip yoxlamaları PHP 7-nin ən səs-küylü xüsusiyyətlərindən biridir. Əslində tip yoxlamaları PHP 5-dən bizə qismən tanışdır. Class, interface-lərə, funksiyalara isə `array` və `callable` kimi yoxlamalar verə bilirdik. Yeni xüsusiyyətə görə aşağıdakı tip yoxlamaları var:

*   **string** - Sətirlər (nümunələr: qələm, məktəb, kitab)
*   **int** - Tam ədədlər (nümunələr: 1, 2 və ya 9)
*   **float** - Onluq kəsr ədədlər (nümunələr: 1.2, 2.5 və ya 5.6)
*   **bool** - Məntiqi dəyərlər (nümunələr: true və ya false)

Susmaya (default) görə PHP 7 zəif tip-yoxlama rejimində işləyir və buna görə də funksiyada ötürülmüş tipi ilk öncə gözlənilən tipə cevirməyə cəhd edəcəkdir və əksər hallarda heç bir səhvlik bildiriş etməyəcəkdir. Artıq bunu da idarə etmək mümkündür. Bunun üçün `<?php` teqindən dərhal sonra `declare(strict_types=1);` elan etmək lazımdır.

> ***Qeyd:** Bu rejim ancaq mövcud fayla təsir edir, cari fayla include olunan digər fayllara və ya bu faylı include edən heç bir fayla aid
> olmayacaqdır.*

Beləliklə bu rejim haqqında öyrəndik ki,
```php
declare(strict_types=0); // zəif tip-yoxlamasını aktiv edir
declare(strict_types=1); // ciddi tip-yoxlamasını aktiv edir
```
Bu rejimləri araşdırmaq üçün hər hansı nümunə bir funksiya yazaq:
```php
function yoxlama(int $i, float $f, string $s, bool $b)
{
    var_dump($i, $f, $s, $b);
}
```
Zəif tip\-yoxlama rejimində, PHP\-nin əvvəlki versiyalarında olan (daxili funksiyalarında) çox zəif xüsusiyyətlərindən biri kimi, tipləri avtomatik konvertasiya etdiyi üçün gözlənilməz nəticələr əldə etmək olar və çox əhəmiyyətli məlumatların itirilməsinə gətirib çıxara bilər. Sadə bir nümunədə ilk arqumentində int tipini tələb edən funksiyada float tipi ötürsək konvertasiya xüsusiyyəti nöqtədən sonra olan hissəni sadəcə kəsib atacaq. Hesab edək ki, zəif tip\-yoxlama rejimi aktivdir.
```php
// İlk olaraq normal parametrlər ötürürük
yoxlama(3, 5.9, 'kitab', true); 
//int(3) float(5.9) string(5) "kitab" bool(true)

// İkinci halda bütün parametrləri yanlış veririk
yoxlama(2.8, 5, true, 10); 
//int(2) float(5) string(1) "1" bool(true)
```
Gördüyümüz kimi hər iki halda funksiya heç bir problemsiz, xəbərdarlıqsız öz işinə davam elədi. 2-ci halda yanlış parametrlər verildiyi üçün isə gözlənilməyən nəticələr verdi.

İndi isə hesab edək ki, ciddi tip-yoxlama rejimi aktivdir `declare(strict_types=1);`
```php
// yanlış parametrləri yenə də ötürməyə cəhd edək.
yoxlama(2.8, 5, true, 10);
Output: Uncaught TypeError: Argument 1 passed to yoxlama() must be of the type integer, float given, called in ....
```
Ciddi yoxlama rejimi hətta tip aliaslarını da qəbul eləmir. Normalda **boolean** tipi üçün true əvəzinə **1**, false əvəzinə **0** verə bilərdiksə, hazırki rejim aktiv olquda bu aliaslar deaktiv olur. Bu isə o deməkdir ki, yuxarıdakı nümunədə birinci parametri düzəltsək, bu dəfə 3-cü arqumentə görə, 3-cünü düzəltsək isə 4-cü arqumentə görə səhvlik verəcək. Yeganə istisna hal kimi **float** tipinə **int** tipini vermək mümkündür. Bu isə məntiqli görünür. Çünki elementar riyaziyyata görə **5** həmişə **5.0** deməkdir. Yəni, istənilən natural ədədi onluq kəsrlə ifadə edə bilərik. 

Növbəti yazı PHP 7-də “funksiyalarda qaytarılan dəyərlərin (return) tiplərinin yoxlaması” və C# dilində mövcud olan çox gözəl bir xüsusiyyətin (operatorun) PHP 7-yə də əlavə olunması haqqında olacaq hansı ki, bu dilə bir az da elitarlıq qatır 😉
