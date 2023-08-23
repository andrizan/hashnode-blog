---
title: "Loging Query Lumen"
datePublished: Wed Aug 23 2023 07:49:59 GMT+0000 (Coordinated Universal Time)
cuid: cllnfpwhc000609jmempc6acc
slug: loging-query-lumen

---

`app\Providers\AppServiceProvider.php`

```php
    function boot()
    {
        app('db')->listen(function ($query) {
            app('log')->info(
                $query->sql,
                $query->bindings,
                $query->time
            );
        });
    
```

log location

`storage\logs\lumen-{date}.log`