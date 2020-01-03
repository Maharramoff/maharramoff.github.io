---
layout: post
title: Laraveldə xüsusi console əmrlərinin yaradılması
date: 2020-01-04 00:56
comments: true
external-url:
categories: Laravel
---

1. İlk olaraq **app/Console/Commands/Make** qovluğunda müvafiq əmr sinfini yaradırıq.
 
`php artisan make:command Make/MakeServiceCommand`

2.  MakeServiceCommand.php faylına aşağıdakı kodları yazırıq

```php
namespace App\Console\Commands\Make;  
      
use Illuminate\Console\GeneratorCommand;  
      
class MakeServiceCommand extends GeneratorCommand  
{  
    protected $name = 'make:service';  
  
    protected $description = 'Create a new service class';  

    protected function getDefaultNamespace($rootNamespace)  
    {  
        return $rootNamespace . '\Services';  
    }  
  
    protected function getStub()  
    {  
        return resource_path('stubs/service.stub');  
    }  
}
```

3. Daha sonra **resources/stubs** qovluğuna **service.stub** faylı əlavə edir və içinə bu kodları yazırıq. Bu bizim generasiya olunacaq Service siniflərimiz üçün şablondur. Laravel Dummy placeholderlərini müvafiq olaraq yuxarıdakı kodda təyin etdiyimiz **DummyNamespace**-i  **getDefaultNamespace** ilə və **DummyClass**-ı isə əmr zamanı daxil edəcəyimiz adla əvəzləyəcək.

```php
namespace DummyNamespace;  
  
class DummyClass  
{  
} 
```

4. Əmr hazırdır. Nümunə olaraq, UserService sinfini yaratmaq üçün konsola aşağıdakı əmri daxil edirik 

`php artisan make:service UserService`

5. Verdiyimiz şablona əsasən aşağıdakı kod generasiya olunacaqdır.

```php
namespace App\Services;  
  
class UserService  
{  
}
```


