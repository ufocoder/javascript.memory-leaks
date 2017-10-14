## Endpoint status

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

