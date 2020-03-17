---
layout: post
title: Laraveldə modellərin istədiyimiz qovluqda yaradılması
date: 2020-01-08 00:15
image: /assets/images/markdown.jpg
headerImage: false
projects: false
tag:
- laravel
- artisan
- php
- model
category: blog
author: Maharramoff
description: Laraveldə modellərin istədiyimiz qovluqda yaradılması
---

>Bildiyimiz kimi, Laravel susmaya görə modelləri `App` namespace-də saxlayır. Kiçik həcmli layihələr üçün bu problem deyil, irihəcmli layihələrlə işləyərkən isə modellərin sayı həddən artıq çox ola bilər və layihə strukturu oxunaqsız, yorucu hala gələ bilər. Bu baxımdan modelləri `App\Models` namespace-nə köçürmək əlverişli ola bilər. 
  
**Nümunə olaraq Post modelinə baxacayıq.**
Post modelini Models qovluğunda yaratmaq üçün, adətən bunu belə bir artisan əmri ilə edə bilərik:  
`php artisan make:model Models/Post` 
  
Hər dəfə qovluğu manual göstərməmək üçün, bu əmri daha da sadələşdirmək olar. Bunun üçün Laravelin kodlarını azca qurcalamaq lazım gələcək :)   
Beləliklə **Illuminate\Foundation\Console** qovluğundan **ModelMakeCommand** sinfini tapırıq və içinə baxırıq.  
```php
namespace Illuminate\Foundation\Console;  
  
use Illuminate\Console\GeneratorCommand;  
use Illuminate\Support\Str;  
use Symfony\Component\Console\Input\InputOption;  
  
class ModelMakeCommand extends GeneratorCommand  
{
    //...kodlar
}
```
  Burada bizə lazım olan metod yoxdur. Daha da dərinə gedərək extend olunan **GeneratorCommand** sinfinə baxırıq və bizə lazım olan  **getDefaultNamespace** metodunu tapırıq.
```php
namespace Illuminate\Console;  
  
use Illuminate\Filesystem\Filesystem;  
use Illuminate\Support\Str;  
use Symfony\Component\Console\Input\InputArgument;  
  
abstract class GeneratorCommand extends Command  
{  
    //...digər kodlar
	
    /**  
    * Get the default namespace for the class. 
    * 
    * @param string  $rootNamespace  
    * @return string  
    */  
    protected function getDefaultNamespace($rootNamespace)  
    {  
        return $rootNamespace;  
    }
	
    //...digər kodlar	
}
```  
Bundan sonrakı işimiz bu metodu override (yenidən təyin etmə) etməkdir.  Bunun üçün, gedirik **App\Console\Commands** qovluğunda **MakeModel** (ad istəyə bağlıdır) adlı bir sinif yaradırıq və Laravelə məxsus **ModelMakeCommand** sinfini extend edərək bizə lazım olan metodu override edirik.

```php
namespace App\Console\Commands;  
  
use Illuminate\Foundation\Console\ModelMakeCommand;  

class MakeModel extends ModelMakeCommand  
{    
    protected function getDefaultNamespace($rootNamespace)  
    {  
        return $rootNamespace.'\\Models'; 
    }  
}
```

Vəssəlam. Artıq, lap başda yazdığımız əmri daha sadə formada yazaraq 

`php artisan make:model Post`

Post modelini təyin etdiyimiz qovluqda yarada bilərik

