## Simple static memory leak

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
