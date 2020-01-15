---
layout: post
title: Sürüşkən nöqtəli ədədlərin ikili say sistemində təsviri
date: 2018-12-06 18:00
image: /assets/images/posts/surusken-noqteli-ededler.png
headerImage: true
tag:
- ieee754
- float
category: blog
author: Maharramoff
description: Sürüşkən nöqtəli ədədlərin ikili say sistemində təsviri
---

## **1.0 IEEE 754 nədir**

IEEE 754–1985 \- 1985\-ci ildə rəsmi olaraq qəbul edilmiş, (2008\-ci ildə yenilənərək IEEE 754–2008 ilə əvəz edilmişdir) kompyuterlərdə sürüşkən nöqtəli ədədləri təmsil edən bir sənaye standartıdır. Hal\-hazırda sürüşkən nöqtəli hesablama əməliyyatları üçün ən çox istifadə edilən format sayılır. IEEE 754–1985 ilk dəfə sınaq kimi Intel 8087 prosessorlarına inteqrasiya edilmişdir.

## **2.0 Eksponensial yazılış və ya Elmi yazılış.**

Eksponensial yazılış (ing. Scientific notation) çox böyük və ya çox kiçik ədədləri ifadə etmək üçün bir düsturdur. Müəyyən riyazi əməliyyatları asanlaşdırdığı üçün əsasən alimlər, riyaziyyatçılar və mühəndislər tərəfindən istifadə olunur. Bu düstura görə istənilən həqiqi ədədi aşağıdakı kimi yazmaq olar:

![](https://miro.medium.com/max/241/1*T4wNPxxsVIS11tssAdj0xw.png)

Şəkil 1

Daha bir\-neçə nümunəyə baxaq:

![](https://miro.medium.com/max/171/1*Hpd519qPPvPGQnJJoPJ2GQ.png)

![](https://miro.medium.com/max/161/1*LEdLoyj1MCDZIxhlfm3P5g.png)

Digər tərəfdən 3.5\-i həm

![](https://miro.medium.com/max/161/1*QmZR_Qh9EI5T1HdQbq6lZg.png)

həm də

![](https://miro.medium.com/max/161/1*mmzishO4PLwmS1ePYTSSwQ.png)

kimi yazmaq olar. Beləliklə, təsvirimiz unikal olmur, nöqtə mantissada irəli və geri “sürüşə” bilir, eksponent isə hər dəfə dəyişir. Bu səbəbdən də, eksponensial yazılışla təqdim edilən ədədlər, həmçinin `“sürüşkən nöqtəli”` ədədlər adlanır.

## 2.1 Normallaşdırılmış forma

Unikal (normallaşdırılmış) formanı mantissada nöqtəni eksponentə müvafiq olaraq irəli və ya geriyə doğru sürüşdürərək, nöqtədən əvvəl yalnız bir ≠ 0 ədəd saxlamaqla əldə etmək olar. Sürüşmələr sola doğru olduqda eksponent müsbət, sağa doğru olduqda isə mənfi olur. Məsələn, şəkil 1\-ə bir də baxaq.

Verilib 125000000 (və ya 125000000.0)
**1.** Nöqtədən əvvəl yalnız bir ≠ 0 ədəd olanadək sürüşmə edirik. **1.25** mantissanı təyin edirik.
**2.** Sola doğru **8** sürüşmə etdik. Demək eksponentimiz +8 olacaq.
**3.** Beləliklə elmi yazılışımız olur: **1.25 x 10⁸**

## **3.0 Sürüşkən nöqtəli ədədlərin IEEE754 ilə** `**təsviri**`

Kompyuterdə onluq ədədlər adətən tam ədədlərdən fərqli təsvir olunur. Səbəbi odur ki, ədədin saxlanması üçün bitlərin yalnız fiks olunmuş bir sayı mövcuddur. IEEE 754 standartına görə əvvəlcə mantissa və eksponenti əks etdirən bitlərin miqdarı təyin edilir. Burada iki variant mövcuddur: 32 bitli sadə format və 64 bitli ikiqat dəqiqlikli format. Hər iki variant eyni metodu istifadə edir. Biz hazırki məqalədə 32 bitlik variantı istifadə edəcəyik. 32 bitlik variantda tam ədədlər `{-2,147,483,648,…, +2,147,483,647}` diapazonunda saxlanıla bilər. Aşağıdakı şəkil\-də bitlərin işarə, mantissa və eksponent üzrə paylanması göstərilmişdir.

![](https://miro.medium.com/max/361/1*Sxl8bzRQBvjTmP8j5ifM5Q.png)

## **3.1 İşarə (1\-bit)**

Bu, sürüşkən nöqtəli ədəddə ilk bitdir və olduqca sadə təyin edilir. Ədəd müsbət olduqda işarəni **0**, mənfi olduqda isə **1** etmək lazımdır.

## 3.2 Eksponent (8\-bit)

8 bitlə biz 0\-dan 255\-ə kimi olan ədədləri göstərə bilərik. `**00000000 = 0**` və `**11111111 = 255**` rezerv edilmişdir; qalmış dəyərlərdən `**{1,…, 254}**` eskponentin faktiki diapazonu `**{-126,…, 127}**` arasında yerdəyişmə (shift) edir. Eksponentin mənfi olmasının mümkünlüyünü nəzərə alaraq, IEEE 754 standartına görə, onun əsl dəyəri aralıq ədəddən alınmalıdır. 8 bit üçün bu aralıq ədəd **127**\-dir. Beləliklə, mənfi işarələrin istifadəsinin mümkün olması üçün biz ekpsonentin üzərinə +127 əlavə etməliyik.

Deyək ki,
\- eksponentimizin 6 olmasını istəyirik. **6 + 127 = 133**, ikili ədədə çeviririk, edir **10000101**
\- eksponentimizin \-8 olması istəyirik. **\-8 + 127 = 119**, ikili ədədə çeviririk, edir **01110111**

## 3.3 Mantissa (23\-bit)

IEEE formatında, sürüşkən nöqtəli ədəd mümkün olduqca normallaşdırılmış qaydada (bax. 2.1) saxlanılır. Normallaşdırılmış qayda mantissanın nöqtədən əvvəl tam bir **≠ 0** ədədi ehtiva etdiyi anlamına gəlir. Lakin ikili sistemdə **≠ 0** olan yalnız bir ədəd var, o da 1\-dir. Buna görə də, normallaşdırılmış ikili mantissa bir qayda olaraq 1 ilə başladığından bu 1\-in saxlanılmasına ehtiyac yoxdur. Faktiki olaraq, IEEE formatında bu 1 ədədi aşkar saxlanmır, ancaq dolayı olaraq nəzərdə tutulur. Beləliklə ikili ədədi elmi yazılışa çevirdikdən sonra, mantissada saxlamazdan əvvəl başlanğıc 1\-i kənarlaşdırırıq. Bu bizə mantissada 1 bit daha çox məlumat saxlamağa imkan verir.

## 3.4 IEEE standartlı ikili say sisteminə çevirmə

Sürüşkən nöqtəli ədədi ikili say sisteminə çevirmək üçün aşağıdakı addımlara əməl etmək lazımdır:

*   **İşarə bitini təyin et** — Ədəd müsbətdirsə 0, mənfidirsə 1 etməli.
*   **Ədədi iki hissəyə ayır** — Tam və kəsr hissəyə ayırmalı.
*   **İkili ədəd çevir** — Hər iki hissəni ikili ədədə çevirməli və nöqtə ilə birləşdirməli.
*   **Eksponenti təyin et** — Alınmış nəticədə nöqtəni ilk 1\-dən sonraya yerləşdirmək üçün neçə addım atılmalı olduğunu təyin etmək lazımdır. Əgər bunun üçün sola sürüşmə edilərsə eksponent müsbətdir, əks halda mənfidir. Alınmış ədədin üzərinə +127 gəlməli və ikili ədədə çevirməli.
*   **Mantissanı normallaşdır** — Bunun üçün nəticədən ilk **1\-i** silməli və sonrakı 23 biti təyin etmək lazımdır.

Bəzi nümunələrə baxaq:

![](https://miro.medium.com/max/558/1*qzC61jx70g6ZUXg67CDQwg.png)

![](https://miro.medium.com/max/582/1*F7vLgtOipcT38EmV-T7ULA.png)

Və yekun olaraq məşhur **0.1 + 0.2 ≠ 0.3** məsələsinə bir də baxaq. Bütün bu yazılanlardan bəlli olur ki, daha sadə dildə desək, kompyuter 0.1\-i əslində 0.1, 0.2 əslində 0.2 kimi saxlamır. Bütün təsvirlər təqribi yaxınlaşmalardır.

![](https://miro.medium.com/max/666/1*BIHkrTlFEX1ttm4wps2sIQ.png)

