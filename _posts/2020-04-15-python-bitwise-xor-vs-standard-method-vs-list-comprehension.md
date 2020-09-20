---
layout: post
title: Python. Bitwise XOR ^ vs Standard method vs List comprehension. Tənha ədədin tapılması və Benchmark.
date: 2020-04-15 14:20
image: /assets/images/posts/python.png
headerImage: true
projects: false

tag:
- python
- benchmark
- bitwise

category: blog
author: Maharramoff
description: Pythonda çox digər dillərdə olduğu kimi Bitwise (bitsəl və ya sadəcə bit operatorları) əməliyyatları mövcuddur. Bitwise operatorları məntiq operatorlarından əsasən ona görə fərqlənir ki, bitsəl operatorlar ədədin bitləri üzərində əməliyyatlar edir. 
---

Pythonda çox digər dillərdə olduğu kimi Bitwise (bitsəl və ya sadəcə bit operatorları) əməliyyatları mövcuddur. Bitwise operatorları məntiq operatorlarından əsasən ona görə fərqlənir ki, bitsəl operatorlar ədədin bitləri üzərində əməliyyatlar edir. Təəssüf ki, bu operatorları çox vaxt düzgün dəyərləndirmirik, bəzi proqramçıların ümumiyyətlə xəbəri olmur praktikası boyunca. İlk baxışdan nəsə mürəkkəb və faydasız kimi görünə bilərlər, amma əslində bu tamamilə belə deyl. Bu misalda kiçik nümunə ilə sadəcə XOR operatoru üzərində bunu isbat etməyə çalışacam.

**Şərt:** List verilib `[2,3,4,3,2,4,5,6,5]`. Listin bütün elementləri birindən başqa cütdür. Daha sadə desək, hər elementdən 2 dənədir, yalnız 1 element təkdir. Bu tənha ədədi tapmalı.

```python
from timeit import repeat as tr
from functools import reduce

L = [2,3,4,3,2,4,5,6,5]

def bitwise_way():
    ret = 0
    for i in L:
	ret ^= i
    return ret
	
def bitwise_lambda_way():
    return reduce((lambda x, y: x ^ y), L)	
	
def standart_way():
    for i in L:
	if L.count(i) == 1: return i
			
def listcomp_way():
    return [i for i in L if L.count(i) == 1][0]
	
print(bitwise_way())	
print(bitwise_lambda_way()) 
print(standart_way()) 
print(listcomp_way()) 
		
print("Bitwise way: ", min(tr(bitwise_way, repeat=10, number=1000000)))
print("Bitwise lambda way: ", min(tr(bitwise_lambda_way, repeat=10, number=1000000)))
print("Standart way: ", min(tr(standart_way, repeat=10, number=1000000)))
print("Listcomp way: ", min(tr(listcomp_way, repeat=10, number=1000000)))

# 6
# 6
# 6
# 6

# Bitwise way:  1.3961095780832693
# Bitwise lambda way:  3.1089426589896902
# Standart way:  4.297130448045209
# Listcomp way:  5.430655234027654
```

Göründüyü kimi 10 raundluq döyüşdən **bitwise_way** digər üsullardan 4-5 dəfə daha sürətli nəticə göstərdi. Niyə belə olduğunu izah edirəm. Bitwise operatorları əslində geniş mövzudur, amma hazırki nümunədə XOR ^ operatoru haqqında məlumat verəcəm.
XOR alqoritminin iş prinsipinə görə verilmiş listdən əslində belə bir return verir 

`2 ^ 3 ^ 4 ^ 3 ^ 2 ^ 4 ^ 5 ^ 6 ^ 5`

Hələ riyaziyyatdan,

```
ab = ba (vuruqların kommutativ qanunu) 
(ab)c = a(bc) (vuruqların assosiativ qanunu) olduğunu bilirik.
```

XOR operatorunun da assosiativ və kommutativ olduğunu bildiyimiz üçün, nəticə əslində belə də ola bilərdi

`(2 ^ 2) ^ (3 ^ 3) ^ (4 ^ 4) ^ (5 ^ 5) ^ 6`

İstənilən 2 eyni ədədin XOR nəticəsi **n ^ n = 0** olduğu üçün, həmçinin **n ^ 0 = n** olduğu üçün belə yaza bilərik. Göründüyü kimi əslində çox sadə əməliyyat baş verir və bu özünü performansda biruzə verir.

`(2 ^ 2) ^ (3 ^ 3) ^ (4 ^ 4) ^ (5 ^ 5) ^ 6 = 0 ^ 0 ^ 0 ^ 0 ^ 6 = 0 ^ 6 = 6`

_XOR ümumiyyətlə necə işləyir ?  **n ^ n** niyə həmişə sıfır verir ?_ 

**XOR ^** operatoru ədədin son bitindən başlayaraq yoxlayır, yalnız hər iki ədədin müvafiq bitləri fərqli olduğu halda 1, qalan hallarda sıfır verir.
Belə bir kod yazaraq alqoritmi tam anlamaq olar

```python
a, b = 25, 26

l = max(a.bit_length(), b.bit_length())

print(a , "^" , b , "\n")
print(a , "\t", bin(a)[2:].rjust(l))
print(b , "\t", bin(b)[2:].rjust(l))
print(a ^ b , "\t", bin(a ^ b)[2:].rjust(l))

'''
Məsələn,

25 ^ 26 = 3

25 	 11001
26 	 11010
3 	    11

Amma olsaydı, 
25 ^ 25

XOR alqoritminə görə nəticə 0 olacaqdı

25 	 11001
25 	 11001
0 	     0

və həmçinin istənilən n ^ 0 niyə həmişə n verir ?

25 ^ 0

XOR alqoritminə yenə baxaq. Ancaq fərqlənən cütlüklər 1 qaytarır, digər hallarda 0. Bir sözlə nəticədə yenə eyni ədəd generasiya olunur.

25 	 11001
0 	 00000
25 	 11001
'''
```

Növbəti məqalələrin birində **n&1** bitwise əməliyyatı ilə ədədin cüt olub olmadığını necə tapırıq, onu izah edəcəm.
