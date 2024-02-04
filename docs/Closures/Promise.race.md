## Not used code

[Details](https://github.com/nodejs/node/issues/17469#issuecomment-685216777)

When you call Promise.race with a long-running promise, the resolved value of the returned promise gets retained for as long as each of its promises do not settle.
Example code to demonstrate:

```js
async function randomString(length) {
    await new Promise((resolve) => setTimeout(resolve, 1));
    let result = "";
    const characters = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789";
    for (let i = 0; i < length; i++) {
        result += characters.charAt(Math.floor(Math.random() * characters.length));
    }
    return result;
}

(async function main() {
    let i = 0;
    const pending = new Promise(() => {});
    while (true) {
        await Promise.race([pending, randomString(10000)]);
        if (i++ % 1000 === 0) {
            const usage = process.memoryUsage();
            const rss = Math.round(usage.rss / (1024 ** 2) * 100) / 100;
            const heapUsed = Math.round(usage.heapUsed / (1024 ** 2) * 100) / 100;
            console.log(`RSS: ${rss} MiB, Heap Used: ${heapUsed} MiB`);
        }
    }
})();
```
In this example, we pass a large random string along with a non-settling promise to Promise.race, and the result is that every single one of these strings is retained.

**How to fix:**
Avoid Promise.race. Replacing your "await Promise.race" expressions with "await new Promise" expressions, where the newly constructed Promise sets a function variable in the outer scope.

```js
(async function main() {
    let i = 0;
    // These functions are set in a promise constructor later
    let resolve;
    let reject;

    const pending = new Promise(() => {});
    pending.then((value) => resolve(value), (err) => reject(err));

    while (true) {
        randomString(10000).then((value) =>  resolve(value), (err) => reject(err));
        // This is the await call which replaces the `await Promise.race` expression in the leaking example.
        // It sets callbacks for our promises
        await new Promise((resolve1, reject1) => {
            resolve = resolve1;
            reject = reject1;
        });
        if (i++ % 1000 === 0) {
            const usage = process.memoryUsage();
            const rss = Math.round(usage.rss / (1024 ** 2) * 100) / 100;
            const heapUsed = Math.round(usage.heapUsed / (1024 ** 2) * 100) / 100;
            console.log(`RSS: ${rss} MiB, Heap Used: ${heapUsed} MiB`);
        }
    }
})();
```
