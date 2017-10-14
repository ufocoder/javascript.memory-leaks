## MeteorJS Example

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

