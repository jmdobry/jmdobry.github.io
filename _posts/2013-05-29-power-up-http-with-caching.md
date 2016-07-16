---
layout: post
title: "Power up $http with caching"
tags: javascript, angular.js, caching
---

- [Intro](#intro)
- [LRU Cache](#lru_cache)
- [Setting a default cache](#setting_a_default_cache)
- [Advanced Caching](#advanced_caching)

# Intro

Angular.js rocks and you're flying high making AJAX requests left and right.
Inject `$http`, shoot off a request and boom—the promise-based service hits your
error or success callback.


```js
$http.get('/foo/bar/' + itemId)
  .success(function (data) {
    data; // { foo: 'bar' }
  })
  .error(function (data, status, headers, config) {
    // uh oh
  });
```

Okay, so you've got your data from your backend—now what? This particular bit of
data doesn't change very often, and you're wondering why your client repeatedly
makes the same request for the same data—what a waste! Save your users some time
and bandwidth and tell that request to use caching!

```js
$http.get('/foo/bar/' + itemId, { cache: true })
  .success(function (data) {
    data; // { foo: 'bar' }
  })
  .error(function (data, status, headers, config) {
    // uh oh
  });
```

Sweet! Now, the first time `$http` sends a `GET` request to `/foo/bar/123`, for
example, the response will be stored in a cache named "$http". This cache is
created by Angular's `$cacheFactory` as the default cache for the `$http`
service when Angular boots up. `{ cache: true }` tells the `$http` service to
cache the response to that particular request in `$http`'s default cache. You
don't have to do anything else. Any subsequent requests to `/foo/bar/123` will
simply use the cached reponse. Neat!

If you want you can actually reference `$http`'s default cache and retrieve
items, remove items, clear the cache, etc. Just inject `$cacheFactory` into
whatever you're working with, and reference `$http`'s default cache via:

```js
var $httpDefaultCache = $cacheFactory.get('$http');
```

You can retrieve the cached response to the above `GET` requested by using the
absolute url of the request.

```js
var cachedData = $httpDefaultCache.get('http://myserver.com/foo/bar/123'); // { foo: 'bar' }
```

You can clear that item from the cache:

```js
$httpDefaultCache.remove('http://myserver.com/foo/bar/123');
```

Or clear the whole cache:

```js
$httpDefaultCache.removeAll();
```

See [$cacheFactory](http://docs.angularjs.org/api/ng.$cacheFactory) for a
complete reference.

# LRU Cache

What if you don't want `$http`'s default cache to store every response? Simple:
turn it into an LRU Cache (Least Recently Used).

```js
// Create a new cache with a capacity of 10
var lruCache = $cacheFactory('lruCache', { capacity: 10 });

// Use the new cache for this request
$http.get('/foo/bar/' + itemId, { cache: lruCache })
  .success(function (data) {
    data; // { foo: 'bar' }
  })
  .error(function (data, status, headers, config) {
    // uh oh
  });
```

The reponse to each request to `/foo/bar/:itemId` will be cached, but this time
the cache only has a capacity of 10. When the eleventh unique request is made
the least recently accessed item will be removed from the caching, keeping the
total number of cached items at 10. This cache maintains a list of its items in
order of how recently they have been accessed so it knows which one to remove
when storing a new item would exceed the capacity.

# Setting a default cache

As shown in the LRU example you can tell an `$http` request to use a cache other
than `$http`'s default cache, but what if you wants to change `$http`'s default
cache? It's easy:

```js
$http.defaults.cache = $cacheFactory('myNewDefaultCache', { capacity: 100 });
```

`$http` will now use `myNewDefaultCache` as its default cache.


# Advanced Caching

What if you want to cache data to improve user experience, but the data is
liable to change once a day, or every few hours, etc. You would want to make
sure your cached data gets cleared at least once a day, every ten minutes, etc.
Unfortunately Angular's `$cacheFactory` does not provide any such capabilities.

You could hack something together using `setInterval()` or `setTimeout()`, but
you don't want to do that. To solve this problem I created [angular-cache](http://jmdobry.github.io/angular-cache/),
a drop-in Angular module that gives you access to `$angularCacheFactory`, which
creates caches with more capabilities. With `angular-cache` your caches can
periodically clear themselves with `cacheFlushInterval`. When adding to a cache
you can specify a `maxAge`, which is the maximum amount of time that particular
item will be in the cache before it is automatically removed. Or you can
specify a `maxAge` for the whole cache which will be applied to every item added
to the cache.

I'll refer you to the angular-cache [documentation](http://jmdobry.github.io/angular-cache/) for further reference.

Happy caching!
