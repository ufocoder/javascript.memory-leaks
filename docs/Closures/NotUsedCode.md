## Not used code

[Source of example](https://bugs.chromium.org/p/chromium/issues/detail?id=315190)

Memory can't be released because it can't be allocated before

```js
function f() {
  var some = [];
  while(some.length < 1e6) {
    some.push(some.length);
  }
  function unused() { some; } //causes massive memory leak
  return function() {};
}

var a = [];
var interval = setInterval(function() {
  var len = a.push(f());
  document.getElementById('count').innerHTML = len.toString();
  if (len >= 500) {
    clearInterval(interval);
  }
}, 10);
```

**How to fix:** investigate and refactor code, try to remove not used functional

```js
function f() {
  return function() {};
}

var a = [];
var interval = setInterval(function() {
  var len = a.push(f());
  document.getElementById('count').innerHTML = len.toString();
  if (len >= 500) {
    clearInterval(interval);
  }
}, 10);
```
