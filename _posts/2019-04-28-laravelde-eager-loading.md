---
layout: post
title: Laravel-də eager loading - with() və bir az da load() haqqında
date: 2019-04-23 00:00
image: /assets/images/posts/eager-loading.png
headerImage: true
projects: false
tag:
- laravel
- php
- eager loading
category: blog
author: Maharramoff
description: Laravel-də eager loading - with() və bir az da load() haqqında
---

Gəlin çox rast gəlinən belə bir situasiyanı təsəvvür edək. Müəllif (Author) və müəllifə aid məqalələr (Post) modelimiz var.
```php
namespace App;

use Illuminate\Database\Eloquent\Model;

class Author extends Model
{
	// Müəllifə aid bütün məqalələr.
    public function posts()
    {
        return $this->hasMany('App\Post');
    }
}
```
Modellərimiz arasında aşağıdakı bağlantılar var
```haskell
Post -> belongsTo -> Author
Author -> hasMany -> Post
```
Müəlliflər və hər müəllifin adı altında ona aid olan postları göstərən belə bir view faylımız var

<!-- {% raw %} -->
```html
@foreach ($authors as $author)
    <h1>Müəllifi {{ $author->name }} olan postlar:</h1>
    <table>
        @foreach ($author->posts as $post)
            <tr>
                <td>{{ $post->title }}</td>
                <td>{{ $post->description}}</td>
            </tr>
        @endforeach
    </table>
@endforeach
```
<!-- {% endraw %} -->

Team Lead qəzəblidir, deadline sıxır, onsuz da gərginlik çoxdur, çaparaq belə bir kod yazırıq və … Həyat davam edir, hər şey gözəldir.

```php
$authors = Author::all();

foreach($authors as $author) 
{ 
    $posts = $author->posts;
}
```
İlk baxışdan kodda heç bir problem yoxdur kimi görünür. Çünki lazım olan məlumatları gözlədiyimiz şəkildə verəcəkdir. Amma gəlin baxaq ayın qaranlıq üzündə nələr baş verir bu zaman.
```sql
-- Bütün müəlliflər Author::all();
SELECT * FROM author;

-- Hər müəllifə aid postlar $author->posts
SELECT * FROM post WHERE author_id = 1
SELECT * FROM post WHERE author_id = 2
SELECT * FROM post WHERE author_id = 3
SELECT * FROM post WHERE author_id = 4
SELECT * FROM post WHERE author_id = 5
-- belə davam
```
Hazırki vəziyyətdə bunun necə səmərəsiz bir üsul olduğuna diqqət yetirin. İndi düşünək ki, 100 müəllifimiz var, müəllifləri almaq üçün 1 sorğu, sonra hər müəllifə aid bütün məqalələri almaq üçün daha bir sorğu göndəririk, nəticədə **100 + 1** sorğu yazmalı olacağıq. Başqa dildə desək məlumat bazasına **N + 1** sorğu göndərmiş oluruq.

> N+1 çox yayılmış bir performans antipatternidir. [Buradan](https://secure.phabricator.com/book/phabcontrib/article/n_plus_one/) tanış ola bilərsiz

Bu zaman səhnədə **with()** metodu ilə **Eager Loading** (*xəsis yükləmə*)  peyda olur.
**with()** metodu \- Eloquent modellərini istifadə edən sorgularla işlədiyimiz zaman həlledici sinif olan **Illuminate\\Database\\Eloquent\\Builder** sinfində elan edilmişdir. “Xəsis yükləmə”, Laravelin, N + 1 problemini həll etmə üsuludur. Bu üsuldan istifadə edərək hazırki nümunədə sorğu sayımızı 2\-yə endirə bilərik. Bunun üçün kodda aşağıdakı dəyişikliyi etmək lazımdır.
```php
$authors = Author::with('posts')->get();

foreach($authors as $author) 
{ 
    $posts = $author->posts;
}
```
`Author::with('posts')->get();` yazarkən biz Laravelə deyirik ki, müəllifləri gətirmişkən, onlara aid məqalələri də gətirdiyinə əmin ol. Bu halda Laravel iki sorğu icra edəcək. “Xəsis” ifadəsi isə burada onu bildirir ki, biz sadəcə bir sorğu ilə modelləri əlaqələndirmiş oluruq. Daha ilkin çoxluğun N elementinə görə N sayda sorğu yazmırıq.

```sql
SELECT * FROM author
SELECT * FROM post WHERE author_id IN (1, 2, 3, 4, 5)
```

Gördüyünüz kimi, Laravelin **Eager Loading** imkanından  istifadə edərək, kodda etdiyimiz kiçik bir dəyişiklik, sorğularımızı daha effektiv şəkildə yerinə yetirməyə imkan verdi.

---

Bir də **load()** metodu ilə **Lazy** **Eager Loading** (Tənbəl xəsis yükləmə) üsuluna baxaq. Bu da çox maraqlı bir metoddur. Bu metod son yazdığımız məntiqi bir növ 2 ayrı mərhələyə ayırır:
**Mərhələ 1** ilkin nəticəni alır.
```php
$authors = Author::all();
//SELECT * FROM author
```

**Mərhələ 2** isə artıq bizim qərarımızı gözləyir. Məsələn, ola bilər ki, məqalələri sonradan müəyyən bir şərt altında göstərmək istəyə bilərik.
```php
if ($condition) 
{
    $authors = $authors->load('posts');
}
//SELECT * FROM post WHERE author_id IN (1, 2, 3, 4, 5)
```

**Sual**
Nə vaxt **`load()`**, nə vaxt **`with()`** istifadə etməli ?
**`load()`** sizə, ikinci sorğunu icra edib etməyəcəyinizi daha sonra, hər hansı dinamik şərtlərə istinadən qərar vermək imkanı yaradır. Lakin əgər bütün əlaqəli elementlərin lazım olacağına əminsinizsə **`with()`** metodundan istifadə edin.

**Yekun**
Heç şübhə etmirəm ki, Eager loading üsulundan əksəriyyət istifadə edir. Daha çox ehtimal edirəm ki, bu üsulun əslində necə işlədiyini bilməyənlər ola bilər. Ümid edirəm ki, bu məqalədən sonra bəzi suallarınıza cavab tapmış oldunuz.
