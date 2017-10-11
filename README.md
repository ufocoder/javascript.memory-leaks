Javascript Memory Leaks Examples
================================

There are examples from [Memory leaks in Javascript](https://slides.com/xufocoder/memory-leaks-in-the-javascript-4) presentation

# Tables of contents

* [Global variables](#global-variables)
  * [Cache service](#cache-service)

## Global variables

### Cache service

There's a cache service with global `cache` variable and after removing this service memory must be released:

```js
(function(window) {
  cache = []

  window.CacheService = function() {
    return {
      cache: function(value) {
        cache.push(new Array(1000).join('*'))
        cache.push(value)
      }
    }
  }
})(window)

var service = new window.CacheService()

for (let i=0; i < 99999; i++) {
  service.cache(i)
}

service = null
```

To fix memory leak we should move `cache` variable to CacheService scope like the followings:

```js
(function(window) {
  window.CacheService = function() {
    var cache = []

    return {
      cache: function(value) {
        cache.push(new Array(1000).join('*'))
        cache.push(value)
      }
    }
  }
})(window)

var service = new window.CacheService()

for (let i=0; i < 99999; i++) {
  service.cache(i)
}

service = null
```
