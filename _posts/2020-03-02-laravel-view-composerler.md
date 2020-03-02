---
layout: post
title: İrəli Səviyyə Laravel 6. View Composerlər.
date: 2020-03-02 21:35
image: /assets/images/markdown.jpg
headerImage: false
tag:
- laravel
- view
- php
category: blog
author: Maharramoff
description: İrəli Səviyyə Laravel 6. View Composerlər.
---

Layihələrlə işləyərkən bəzən elə datalar olur ki, bu eyni datanı bir çox və ya hətta bütün səhifələrdə istifadə etmək gərəkir. Bu cür hallar üçün Laravel çox eleqant bir həll kimi View Composerləri təqdim edir. 
Laravelin rəsmi dokumentasiyası bunu belə izah edir:

> View Composerlər, şablonların render edilıdiyi zaman icra edilən
> callback funksiyaları və ya sinif metodlarıdır. Hər hansı bir datanı,
> müəyyən bir və ya bir neçə şablona göndərmək istəyirsinizsə, bu halda
> View Composerlər bu cür məntiqləri bir yerdə orqanizə etməyə kömək edə
> bilərlər. 

Bunu anlamaq üçün real bir nümunəni nəzərdən keçirək və müxtəlif həll variantlarına nəzər yetirək. Hesab edək ki, müştərilərlə iş üçün hər hansı bir CRM layihəmiz var və bu müştərilərin siyahısı bizə sistemin bir çox səhifələrində lazım olur. Bunlardan biri **müştəridən ödəniş qəbul etmə** digəri isə **müştərilərin siyahısı** səhifəsi olsun.  Birində datanı `select`-ə, digərində isə müştəri linkləri siyahısına ötürəcəyik. Müvafiq olaraq `Customer` modelimiz, `PaymentController`, `CustomerController`  kontrollerlərimiz və aşağıdakı marşrutlarımız var:

```php
Route::get('payment/create', 'PaymentController@create');  
Route::get('customer', 'CustomerController@index');
```

**Həll 1.** İlk olaraq ənənəvi üsulla həll edək. Hər iki kontrollerimizi yaradırıq və içinə bu kodları yazırıq. 
```php
namespace App\Http\Controllers;  
  
use App\Customer;  

class PaymentController extends Controller  
{  
    public function create()  
    {  
        $customers = Customer::all();  
        
        return view('payment.create', compact('customers'));  
    }  
}
```
və
```php
namespace App\Http\Controllers;  
  
use App\Customer;  

class CustomerController extends Controller  
{  
    public function index()  
    {  
        $customers = Customer::all();  
        
        return view('customer.index', compact('customers'));  
    }  
}
```
Daha sonra view-ları yaradaq.
1. payment.create

```php
<form>
    <select name="customer_id" id="customer_id">   
        @foreach($customers as $customer)  
            <option value="{{ $customer->id }}">{{ $customer->name }}</option>  
        @endforeach  
    </select>
</form>
```
2. customer.index
```php
<div class="content">
    <ul>   
        @foreach($customers as $customer)  
            <li><a href="customer/{{ $customer->id }}">{{ $customer->name }}</a></li>  
        @endforeach  
    </ul>
</div>
```
Okay. İlk baxışdan tamamilə normal bir həll kimi görünür. Amma burada bir problem gizlənir. Növbəti həll metodunda bunu açıqlayacaqam.

**Həll 2.** `View::share` metodu.
İndi isə təsəvvür edin ki, tələb gəlir ki, bütün sistem üzrə müştərilərin siyahısı mütləq adlara görə sıralanmalıdır. Yuxarıdakı həll variantımızda biz bunun üçün hər bir controllerdə belə bir dəyişiklik etməli olardıq:
```php
class PaymentController extends Controller  
{  
    public function create()  
    {  
        $customers = Customer::orderBy('name')->get();  
        
        return view('payment.create', compact('customers'));  
    }  
}

class CustomerController extends Controller  
{  
    public function index()  
    {  
        $customers = Customer::orderBy('name')->get();  
        
        return view('customer.index', compact('customers'));  
    }  
}
```
Hazırda 2-dir, amma daha sonra bu eyni datanı və ya hər hansı digər bunun kimi digər bir datanı sistemin 4-5 və daha çox yerlərində istifadə etmək lazım olarsa necə ? Biz demək hər dəfə öncə bütün fayllarda axtarış etməli, bir-bir hər birində dəyişiklik etməli olarıq. Problemin nə olduğu indi yəqin ki aydın olur. Bunun nisbətən daha yaxşı həll variantı kimi View fasadının share metodundan istifadə edək. `AppServiceProvider`-da boot metoduna belə bir kod yazırıq.

```php
namespace App\Providers;  

use App\Customer;  
use Illuminate\Support\Facades\View;  
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider  
{  
    public function boot()  
    {  
        View::share('customers', Customer::orderBy('name')->get());  
    }  
}
```
Bu metodla biz sistem üzrə bütün view fayllarında `$customers` datasını istifadə edə bilərik. Ona görə də artıq controllerlərdən bizə lazım olmayan kodları silmək olar. View-larda isə faktiki olaraq, heç bir dəyişikliyə gərək yoxdur.

```php
class PaymentController extends Controller  
{  
    public function create()  
    {  
        return view('payment.create');  
    }  
}

class CustomerController extends Controller  
{  
    public function index()  
    {  
        return view('customer.index');  
    }  
}
```
Buna bəlkə də çox yaxşı həll kimi baxmaq olar, amma nəzərə alsaq ki, çox nadir hallarda eyni datanı bütün sistem üzrə istifadə edirik, yazdığımız həllə görə istənilən bir view fayl render olunan zaman məlumat bazasına hər dəfə gərəkli olub-olmadan sorğu göndərmiş olacayıq. 3-cü həlldə buna baxaq.

**Həll 3.** `View::composer` metodu. Bu metod bizə imkan verir ki, datanı məhz bizə lazım olan view-lar arasında paylaşaq. Hazırki nümunədə bizə bu datanı sadəcə 2 view üçün paylaşmaq istəyirik. Odur ki, composer metoduna aşağıdakı parametrləri ötürməliyik.

```php
class AppServiceProvider extends ServiceProvider  
{  
    public function boot()  
    {  
        View::composer(['payment.create', 'customer.index'], function ($view)  
        {  
            $view->with('customers', Customer::orderBy('name')->get());  
        });
    }  
}
```
Bununla məqaləni bitirmək olardı əslində. Amma gəlin bir az da dərinə getməyə çalışaq. Məqalənin bundan sonrakı hissəsində SOLID, DRY, KISS prinsiplərinə mümkün qədər əməl etməyə çalışacayıq. 

**Həll 3.1.** Son yazdığımız bir sətirli həll əksər hallarda kifayət edər. Amma bəzən bir datanı əldə etmək üçün, çoxsaylı əməliyyatlar, yoxlamalar, şərtlər yazırıq. Bu qədər kodu boot metodunda yazmaqdansa, onu özünə məxsus bir sinfə köçürmək olar.  Bunun üçün `app/Http/View/Composers` direktoriyası (adlandırma və yerləşmə sərbəstdir) yaradaq və oraya `CustomersComposer` sinfini əlavə edək.
```php
namespace App\Http\View\Composers;  
  
use Illuminate\View\View;  
  
class CustomersComposer  
{  
    public function compose(View $view)  
    {  
        $view->with('customers', Customer::orderBy('name')->get());  
    }  
}
```
`AppServiceProvider`-də müvafiq dəyişikliyi edək.
```php
use App\Http\View\Composers\CustomersComposer;

class AppServiceProvider extends ServiceProvider  
{  
    public function boot()  
    {  
        View::composer(['payment.create', 'customer.index'], CustomersComposer::class);
    }  
}
```
**Həll 3.2.** Nəzərə alsaq ki, Laravelin imkanları çox genişdir, bir az da dərinə gedə bilərik. Gəlin, həll 1-də yazdığımız view fayllarını bir qədər də optimallaşdıraq. Düşünün ki, select elementini və ya digər listi eynilə başqa view fayllarında da istifadə etmək istəyirik. Buna görə biz yenə yuxarıdakı addımları təkrarlamalı olarıq, view fayl yaratmalı, həmin view faylı AppServiceProvider-də qeyd etməli olarıq. Bu qədər təkrarçılığa yol verməmək üçün, `resources/views/partials/customers` direktoriya strukturunu yaradaq və içinə belə bir blade faylları əlavə edək.
Bunlardan biri `select.blade.php` olsun
```php
<select name="customer_id" id="customer_id">   
    @foreach($customers as $customer)  
        <option value="{{ $customer->id }}">{{ $customer->name }}</option>  
    @endforeach  
</select>
```
Digəri olsun `list.blade.php`
```php
<ul>
    @foreach($customers as $customer)  
        <li><a href="customer/{{ $customer->id }}">{{ $customer->name }}</a></li>  
    @endforeach  
</ul>
```
Blade-in `@include` direktivi bizə bir blade faylını digər blade fayllarında çağırmağa imkan verir. Bundan yararlanaraq **Həll 1**-dəki fayllarda aşağıdakı dəyişiklikləri edək. 

1. payment.create
```php
<form>
	@include('partials.customers.select')
</form>
```
2. customer.index
```php
<div class="content">
	@include('partials.customers.list')
</div>
```
`AppServiceProvider`-də isə bu dəfə asterisk `*` simvolundan istifadə edərək `Customers` datasını `partials.customers` qovluğunda olan bütün fayllara göndərə bilərik.
```php
use App\Http\View\Composers\CustomersComposer;

class AppServiceProvider extends ServiceProvider  
{  
    public function boot()  
    {  
        View::composer('partials.customers.*', CustomersComposer::class);
    }  
}
```
Bundan sonra həmin datanı daha başqa formada başqa bir faylda istifadə etmək lazım olduqda artıq çoxsaylı faylları dəyişməyə ehtiyac olmayacaq. Sadəcə `partials.customers` qovluğunda sizə lazım olan blade faylını yaradacaq və onu istifadə etmək istədiyiniz faylda `@include` edəcəksiniz. Bu qədər sadə.

**Bonus.** Əgər sizə lazım olan data çox dinamik bir data deyilsə və sistemin çox hissəsində istifadə olunursa, Laravelin cache imkanından istifadə edərək həllimizi daha da optimallaşdıra bilərik. Bunun üçün `CustomersComposer`-də aşağıdakı dəyişikliyi edək.

```php
namespace App\Http\View\Composers;  
  
use Illuminate\View\View;
use Illuminate\Support\Facades\Cache;

class CustomersComposer  
{  
    public function compose(View $view)  
    {  
        $cacheTimeToLive = 3600;
        $customers = Cache::remember('customers', $cacheTimeToLive, function ()
        {
            return Customer::orderBy('name')->get();
        });
		
        $view->with('customers', $customers);  
    }  
}
```

