---
layout: post
title: Python. Listdən onun elementlərinin təkrarlanma tezliyini dict ilə çıxarmalı + Benchmark
date: 2018-01-11 15:25
image: /assets/images/posts/python.png
headerImage: true
projects: false

tag:
- python
- benchmark

category: blog
author: Maharramoff
description: Verilmiş listdən onun elementlərinin təkrarlanma tezliyini dict ilə çıxarmalı. Normal və bəzi anormal üsullar.
---

Verilmiş listdən onun elementlərinin təkrarlanma tezliyini dict ilə çıxarmalı. Normal və bəzi ~~anormal~~ üsullar. Sonda benchmark mikrooptimizasiya sevənlər üçün.

**Şərt:** `['bmw', 'skoda', 'bmw', 'ferrari', 'audi', 'audi', 'skoda', 'bmw', 'skoda', 'bmw']` listi verilib. `{'bmw': 4, 'skoda': 3, 'ferrari': 1, 'audi': 2}` nəticəsini əldə etməli. Sıralama önəm daşımır.

```python
from collections import defaultdict as df, Counter as cn
from numpy import unique as nu
from timeit import repeat as tr

CARS = ['bmw', 'skoda', 'bmw', 'ferrari', 'audi', 'audi', 'skoda', 'bmw', 'skoda', 'bmw']

def standart_way():
    d = {}
    for car in CARS:
        if car not in d:
            d[car] = 0
	d[car] += 1
    return d
	
def count_way():
    d = {}
    for car in CARS:
	d[car] = CARS.count(car)
    return d	
 
def get_way():
    d = {}
    for car in CARS:
	d[car] = d.get(car, 0) + 1
    return d    

def defaultdict_way():
    d = df(int)
    for car in CARS:
	d[car] += 1
    return d 

def counter_way():
    return cn(CARS)
	
def numpy_way():
    unique, counts = nu(CARS, return_counts=True)
    return dict(zip(unique, counts))
	
print("Standart Way: ", min(tr(standart_way, repeat=10, number=1000000)))
print("Count Way: ", min(tr(count_way, repeat=10, number=1000000)))
print("Get Way: ", min(tr(get_way, repeat=10, number=1000000)))
print("Defaultdict Way: ", min(tr(defaultdict_way, repeat=10, number=1000000)))
print("Counter Way: ", min(tr(counter_way, repeat=10, number=1000000)))
print("Numpy Way: ", min(tr(numpy_way, repeat=10, number=1000000)))

'''
Standart Way:  4.347476382012246
Count Way:  9.187473141006194
Get Way:  4.505646449979395
Defaultdict Way:  4.317712290008785
Counter Way:  10.286595062993001
Numpy Way:  46.717036856978666
'''
```

Göründüyü kimi millisaniyə fərqi ilə defaultdict-ə uduzsa da, standart üsul kifayət qədər optimal nəticə verir.  Demək heç bir əlavə modul import etməyə gərək yoxdur.

