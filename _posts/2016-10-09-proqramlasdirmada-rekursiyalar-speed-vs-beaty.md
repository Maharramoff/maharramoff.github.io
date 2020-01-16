---
layout: post
title: Proqramlaşdırmada Rekursiyalar. "Speed vs Beauty"
date: 2016-10-09 00:00
image: /assets/images/posts/fibo-recursion.jpg
headerImage: true
tag:
- sürət
- rekursiya
- c++
category: blog
author: Maharramoff
description: Proqramlaşdırmada Rekursiyalar. "Speed vs Beauty"
---

**Rekursiyadan** istifadə proqram kodunun həcmini azalda bilər, kodu daha qısa və aydın edə bilər. Amma rekursiyanın da öz çatışmazlıqları var.

Nümunə üçün [fibonaççi ədədlərinin](https://www.wikiwand.com/az/Fibona%C3%A7%C3%A7i_%C9%99d%C9%99dl%C9%99ri?fbclid=IwAR0HZf7q70zEAXPO7P90-v_rsOE8VCP1NQz6QIsObi4rhNVnQOTvg3FLJK0) hesablanmasına baxaq. İlk baxışdan çox sadə və sürətli işləyəcək kimi görünür.
```cpp
long fibo (int n)
{     
    return n <= 1 ? n : fibo(n - 1) + fibo(n - 2);
}
fibo(45);  
// artıq 45-ci həddin hesablanmasında işləmə sürəti ~ 9.08 saniyə
```

Bu funksiyada N-ci Fibonaççi sayının tapılması üçün hər çağırış daha iki rekursiv çağırış edir. Bu isə o deməkdir ki, əməliyyatın müddəti aşağıdakı şəkil-dən göründüyü kimi geometrik ölçüdə böyüyür. Diqqətlə baxsaq görərik ki, **F(6)** hesablanması üçün **F(3)** üç dəfə hesablanır. Belə çıxır ki, **F(n)** hesablanmasına n-in böyük dəyərlərində baxsaq, onda lap çox təkrar hesablamalar olacaq. Eyni dəyərlərin təkrar hesablanması məhz Rekursiyanın əsas çatışmazlığıdır.

![](/assets/images/posts/fibo-recursion.jpg)

Bu funksiyanı optimallaşdırmaq üçün, artıq sayılmış dəyərləri hansısa şəkildə saxlamaq olar. Məsələn, aşağıdakı həldə rekursiyanın yerinə iterasiyadan istifadə edilmişdir.
```cpp
long fibo_yeni (int n)
{ 
    long f_1 = 1, f_2 = 0, f_hazirki; 
    if (n > 1) 
    { 	
        while (--n) 
        {            
            f_hazirki = f_1 + f_2; 
            f_2 = f_1; 
            f_1 = f_hazirki; 
        }   
        
        return f_hazirki;
    }
    
    return n;
}  
fibo_yeni(10000000); 
//hətta belə böyük bir həddin hesablanma sürəti max. ~ 0.04 saniyə
```
Rekursiv alqoritmlər yaddaşa çox tələbkardır. Hər rekursiv çağırış yeni stek yaradır. Məsələn, ilk funksiyamıza baxsaq görərik ki, nə qədər böyük sayı hesablasaq, bir o qədər də böyük yaddaş tələb olunacaq. İstənilən rekursiv alqoritmi iterasiya ilə həll etmək olar, hərçənd ki, bu üsul adətən daha mürəkkəb və həcmli olur.

**Nəticə: Qısa və gözəl görünən kod həmişə optimal kod deyil.**
