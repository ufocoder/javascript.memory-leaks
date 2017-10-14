Javascript Memory Leaks Examples
================================

Example of Javascript Memory leaks with explanations and fixes

There is an presentation about [Memory leaks in Javascript](https://slides.com/xufocoder/memory-leaks-in-the-javascript-4)

# Tables of contents

* [Global variables](./docs/GlobalVariables/README.md)
  * [Cache service](./docs/GlobalVariables/CacheService.md)
* [Callbacks](#callbacks)
  * [Endpoint status](#endpoint-status)
* [Not killed timers](#not-killed-timers)
  * [Gonzalo Ruiz de Villa Modified Example](#gonzalo-ruiz-de-villa-modified-example)
* [Closures](#closures)
  * [Not used code](#not-used-code)
  * [MeteorJS Example](#meteorjs-example)
* [DOM References](#dom-references)
  * [Simple static memory leak](#simple-static-memory-leak)

## Global variables

Scripts create objects into global scope 

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


## Callbacks

### Endpoint status

**What is a memory leak:** interval create event listeners for every tick

```js
function checkStatus() {
  fetch('/endpoint').then(function(response) {
    var container = document.getElementById("container"); 
    container.innerHTML = response.status;

    container.addEventListener("mouseenter", function mouseenter() { 
      container.innerHTML = response.statusText;
    });
    container.addEventListener("mouseout", function mouseout() { 
      container.innerHTML = response.status;
    });
  })
}

setInterval(checkStatus, 100);
```

**How to fix:** extract creating event listeners 

```js
var container = document.getElementById("container");
var status = {
  code: null,
  text: null
}

container.addEventListener("mouseenter", function() { 
  container.innerHTML = status.code;
});

container.addEventListener("mouseout", function() { 
  container.innerHTML = status.text;
});

function processResponse(response) {
   status.code = response.status
   status.text = response.statusText
}

function checkStatus() {
  fetch('/endpoint').then(processResponse)
}

setInterval(checkStatus, 100)
```

## Not killed timers

### Gonzalo Ruiz de Villa Modified Example

[Source of example](http://slides.com/gruizdevilla/memory#/5/17)

```js
var strangeObject = {
  storage: [], 
  callAgain: function () {
    var ref = this 
    ref.storage.push(new Array(1000000).join('*'))
    var val = setTimeout(function () {
      ref.callAgain() 
    }, 50) 
  } 
} 

strangeObject.callAgain() 
strangeObject = null
```

**How to fix:** use interval and allow to clean it

```js
var strangeObject = {
  storage: [], 
  startCallAgain: function() {
    this.interval = setInterval(this.tickCallAgain.bind(this), 50) 
  },
  tickCallAgain: function() {
    this.storage.push(new Array(1000000).join('*'))
  },
  stopCallAgain: function() {
    if (this.interval) {
      clearInterval(this.interval)
    }
  }
} 

strangeObject.startCallAgain() 

setTimeout(function() {
  strangeObject.stopCallAgain()
  strangeObject = null
}, 5000)
```

## Closures

### Not used code

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

### MeteorJS Example

[Source of example](https://blog.meteor.com/an-interesting-kind-of-javascript-memory-leak-8b47d2e7f156) 

```js
var theThing = null
var replaceThing = function () {
  var originalThing = theThing
  var unused = function () {
    if (originalThing)
      console.log("hi")
  }
  theThing = {
    longStr: new Array(1000000).join('*'),
    someMethod: function () {
      console.log(someMessage)
    }
  }
}
setInterval(replaceThing, 1000)
```

**How to fix**

```js

var theThing = null
var replaceThing = function () {
  var originalThing = theThing
  var unused = function () {
    if (originalThing)
      console.log("hi")
  }
  theThing = {
    longStr: new Array(1000000).join('*'),
    someMethod: function () {
      console.log(someMessage)
    }
  }
  originalThing = null;
}
setInterval(replaceThing, 1000)
```


## DOM References

### Simple static memory leak

```html
<div id="container">
  <form>
    <input type="submit" value="Push me" />
  </form>
</div>
```

```js
var elements = {
  container: document.querySelector('#container'),
  form: document.querySelector('form'),
  submit: document.querySelector('[type="submit"]')
}

elements.form.addEventListener('submit', function(e) {
  e.preventDefault()
  elements.container.innerHTML = 'Ops..'
})
```

**How to fix**

```js
var elements = {
  container: document.querySelector('#container'),
  form: document.querySelector('form'),
  submit: document.querySelector('[type="submit"]')
}

function processSubmit(e) {
  e.preventDefault()

  elements.form.removeEventListener('submit', processSubmit)
  elements.container.innerHTML = 'Ops..'
  
  elements = {
    container: document.querySelector('#container')
  }
}

elements.form.addEventListener('submit', processSubmit)
```
