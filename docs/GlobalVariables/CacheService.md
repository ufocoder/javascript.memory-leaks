### Cache service

**What is a memory leak:** after removing `CacheService` instance memory must be released 

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

**How to fix:** move `cache` variable to `CacheService` scope like the followings:

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
