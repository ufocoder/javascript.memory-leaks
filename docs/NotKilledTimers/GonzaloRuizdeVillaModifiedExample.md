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
