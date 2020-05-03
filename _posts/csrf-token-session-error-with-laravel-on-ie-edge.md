---
layout: page
title: Calculate distance between two latitude – longitude points using MySQL.
description: The problem was with storing the cookies on Internet Explorer and Edge browsers. After doing a lot of search, I came to understand that the issue is basically with P3P Policy.
permalink: /csrf-token-session-error-with-laravel-on-ie-edge/
---

# CSRF tokenMismatch Exception / Session Error with Laravel on IE/Edge.

Recently, I used laravel 5.2 for a project and faced a weird error with handling cookies. The problem was with storing the cookies on Internet Explorer and Edge browsers. After doing a lot of search, I came to understand that the issue is basically with P3P Policy.

You could get a lot of answers if you do a search, but I got confused about how to implement those solutions. At last I was able to fix the issue using middleware, and thought of sharing the solution with the world.

All you need to do is create a middleware and include the same in all routes,

```php
php artisan make:middleware IeFix
```

Above command will create a new middleware in the `app/Http/Middlewares` directory named `IeFix.php`. Now, we have to update the `handle` function in the `IeFix.php`.

```php

public function handle($request, Closure $next)
{
    $response = $next($request);
    $response->header('P3P', 'CP="IDC DSP COR ADM DEVi TAIi PSA PSD IVAi IVDi CONi HIS OUR IND CNT"');
    return $response;
}

```

And then, update the `Kernal.php` in `app/Http/` to register the middleware. You may either include the new middleware in Laravel's default middleware group `web` or add to route middleware and include in routes.

### 1. Updating `web` middleware will look like,

```php
protected $middlewareGroups = [
  'web' => [
      \App\Http\Middleware\EncryptCookies::class,

      // ..At the end of list....
      \App\Http\Middleware\IeFix::class,
  ],
  'api' => [
    'throttle:60,1',
  ],
];
```

### 2. And the other option, you can just add a new entry in `$routeMiddleware` array like,

```php
protected $routeMiddleware = [
  'auth' => \App\Http\Middleware\Authenticate::class,
  'iefix' => \App\Http\Middleware\IeFix::class
];
```

and then include this middleware in routes like,

```php
Route::group(['middleware' => ['web','iefix'],'prefix'=>'admin'], function () {
     // your routes here
});
```

That's It!

### References

[https://github.com/laravel/framework/issues/2962#issuecomment-53048718](https://github.com/laravel/framework/issues/2962#issuecomment-53048718)
