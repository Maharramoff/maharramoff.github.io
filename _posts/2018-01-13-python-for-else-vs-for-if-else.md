---
layout: post
title: Python. For..else VS For, if..else
date: 2018-01-13 20:35
image: /assets/images/posts/python.png
headerImage: false
projects: false

tag:
- python

category: blog
author: Maharramoff
description: Python. For..else VS For, if..else
---

**for** ilə birgə **else** şərtindən istifadə edən çox az adam tapılar. Çünki bu, python dilinin spesifik konstruksiyalarından biridir və əksər proqramçılar hələ də digər dillərdən xatırladığı (C, C++, Java, PHP və s.) üsulları tətbiq edirlər. Amma məlumunuz olsun ki, loopların body-sində break açar sözü istifadə olunduğu əksər hallarda else şərti çox yerinə düşür.

**else** necə işləyir? Çox sadə. Bu seksiyanın kodu yalnız o zaman yerinə yetirilir ki, əsas dövr təbii olaraq sona çatır, yəni heç bir exception və ya break çağırışı olmursa.

**şərt**: Verilmiş aralıqda 113 və 117-yə qalıqsız bölünən ədədin olub olmadığını yoxlamalı.

```python
from timeit import repeat as tr

def standart_way():
    found = False
    for i in range(1, 13221):
        if i % 113 == 0 == i % 117: 
	    found = True
	    break
    if found:
	return "Found!"
    else:
	return "Not Found!"

def for_else_way():
    for i in range(1, 13221):
	if i % 113 == 0 == i % 117:
            return "Found!"
	    break
    else: # break olmadısa, seksiya işə düşür.
        return "Not Found!"
		
# print(standart_way())
# print(for_else_way())

print("Standart Way: ", min(tr(standart_way, repeat=10, number=1000)))
print("For Else Way: ", min(tr(for_else_way, repeat=10, number=1000)))

# Not Found! if range(1, 13221)
# Standart Way:  1.9296428450034
# For Else Way:  2.5060362959629856


# Found! if range(1, 13222)
# Standart Way:  1.9911524420022033
# For Else Way:  1.925913753977511
```

Nəticə o qədər də xoşbəxt etmədi ) Amma yenə də bu cüzi fərq ən azı 3 sətir daha az və daha gözəl kod yazmaqdan imtina üçün səbəb ola bilməz. Bundan başqa ədədin tapıldığı və tapılmadığı hallarda nəticələr də fərqli oldu qəribə də olsa. Amma hədəfimiz məhz else seksiyası olduğu üçün benchmarkda standart üsul ~0.6 saniyə ilə qalib oldu.

**p.s.** Yeri gəlmişkən, koda diqqət etdinizsə `== 0 ==` üsulu müqayisə yazmışam, amma adətən digər dillərdən vərdiş olaraq belə yazırıq `if i % 113 == 0 and i % 117 == 0:`  Bu da Pythonu fərqli edən xüsusiyyətlərindən daha biridir 😉

