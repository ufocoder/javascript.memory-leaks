## prevObject

**What is a memory leak:** jQuery supports chaining/fluid style. In order to make it work it keeps references to other jQuery objects in a `.prevObject` property

```js
const $leaky = $('<div><span>LEAK</span></div>').find('span').parent().empty(); // [ <div></div> ]

// <span> has been removed from the <div> and we expect it to be garbage collected

// Oh no! It's still here!
console.log( $leaky.prevObject ); // [ <span>LEAK</span> ]

// And we can "end" the chaining and get it back even if it's already detached
console.log( $leaky.end() ); // [ <span>LEAK</span> ]
```

**How to fix:** Delete the `.prevObject` or create a new jQuery object from this one

```js
const $clean = $(
    $('<div><span>LEAK</span></div>').find('span').parent().empty()
); // [ <div></div> ]

// <span> has been removed from the <div> and we expect it to be garbage collected

// Yup, there's no more references and it can be GC'ed.
console.log( $clean.prevObject ); // undefined
console.log( $clean.end() ); // [ /* an empty jQuery object */ ]
```
