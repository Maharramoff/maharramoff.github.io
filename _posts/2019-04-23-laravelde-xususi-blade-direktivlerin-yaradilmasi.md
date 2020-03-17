---
layout: post
title: Laraveldə xüsusi Blade direktivlərin yaradılması
date: 2019-04-23 00:00
image: /assets/images/posts/blade-directives.png
headerImage: true
projects: false
tag:
- laravel
- php
- blade
category: blog
author: Maharramoff
description: Laraveldə xüsusi Blade direktivlərin yaradılması
---

Adətən Blade view fayllarında hər hansı rol/access və ya bunun kimi digər çox rast gəlinən yoxlamalar bu şəkildə edilir.
```php
@if(Auth::check() && Auth::user()->role['name'] === 'admin')
   // Bu hissəni ancaq admin görür
@endif
```

İndi isə təsəvvür edin layihə genişlənib, rollar və ya yoxlamalar çoxalıb, müvafiq olaraq view fayllar da artıb. Bu qədər kodun view fayllarında olması elə də yaxşı praktika sayılmır. Bunun qarşısını almaq və daha eleqant/oxunaqlı kod yazmaq üçün Laravelin xüsusi blade direktivlərindən istifadə etmək olar. İlk olaraq blade service yaradırıq. Bunu **AppServiceProvider**-də də bind etmək olar. Amma daha yaxşı təcrübə baxımından ayrı yaratmağı tövsiyə edirəm.

**Step 1.** Aşağıdakı əmrlə **BladeServiceProvider** yaradırıq. Ad seçimi sərbəstdir. (BladeProvider də ola bilər və s.)

`php artisan make:provider BladeServiceProvider`

**Step 2.** Yaratdığımız servisi **config/app.php** massivində müvafiq yerdə qeydiyyatdan keçiririk.

```php
'providers' => [
    ...
    App\Providers\BladeServiceProvider::class,
],
```
**Step 3.** Servisimizi **IoC** konteynerdə qeyd etdikdən sonra özəl blade məntiqlərimizi yaza bilərik. Bundan sonra **app/Providers** qovluğunda həmin faylı açırıq və register metodunda belə bir kod yazırıq. Nümunədə **isadmin** direktivi yazırıq, yenə də adlandırma seçimi sərbəstdir.


```php
<?
// Laravel v >= 5.5 isə
public function register()
{
    \Blade::if('isadmin', function ()
    {
        return Auth::check() && Auth::user()->role['name'] === 'admin';
    });
}

// Əks halda köhnə üsulla yazmalıyıq
private function register()
{
    \Blade::directive('isadmin', function ()
    {
        return "<?php if (Auth::check() && Auth::user()->role['name'] === 'admin'): ?>";
    });

    \Blade::directive('endisadmin', function ()
    {
        return '<?php endif; ?>';
    });
}
?>
```

**Step 4.** Direktivləri yazdıqdan sonra, ümumiyyətlə direktivlərdə nə zaman kiçik də olsa dəyişiklik edilərsə, dərhal bütün keşlənmiş view fayllar aşağıdakı əmrlə təmizlənməlidir.

`php artisan view:clear`

**Final Step.** Yazdığımız direktivləri test edirik.

```php
@isadmin
// Bu hissəni ancaq admin görür...
@endisadmin
```
**UPD.** Blade compiler normalda özəl yazılmış direktiv aşkarlayarsa tək arqumentli xüsusi callback qaytarır. **Step 3**-dəki nümunəyə əlavə olaraq, əgər rollarımız çox olarsa məsələn, manager, writer və s. kimi, bu halda hər biri üçün ayrı direktiv yazmaqdansa callback-dan istifadə edərək daha effektiv bir Closure yaza bilərik.

```php
<?
public function register()
{
    \Blade::if('roleis', function($role_name)
    {
        return Auth::check() && Auth::user()->role['name'] === $role_name;
    });
}
?>
```

Bu halda Blade faylında yoxlamamız bu şəkildə olacaq

```php
@roleis('admin')
// Bu hissəni ancaq admin görür
@endroleis 

@roleis('manager')
// Bu hissəni ancaq menecer görür
@endroleis
```

**Nəticə.** Xüsusi direktivlər imkanından istifadə edərək view fayllarımızı daha səliqəli, yığcam və oxunaqlı hala sala bilərik. Ən əsası isə yoxlama məntiqini nə zamansa dəyişmək lazım olarsa bunun üçün sistemdə olan 10-larla bəzən 100-lərlə view faylında dəyişiklik etməkdənsə, sadəcə direktivdən həmin məntiqi dəyişmək kifayət edəcək. Bununla da özümüzü, həmçinin özümüzdən sonra kodlarımıza baxacaq, istifadə edəcək proqramçıları da happy etmiş olarıq.
