# async/await workshop
Antonio Valverde

---

Slides URL:

https://gitpitch.com/toniov/async-await-workshop

---

# Profile

![](assets/imgs/profile.jpg "Sakamoto")

- Server side engineer
- Working at [CyberAgent, Inc](https://www.cyberagent.co.jp/en/)
- Currently working with Node.js (ES2017 usage)

---

This workshop can be followed using the Google Chrome console:

- Windows / Linux: `Ctrl` + `Shift` + `J`
- Mac: `Cmd` + `Opt` + `J`

---

The following function will be used in the examples in this workshop:

```js
const delay = (ms) => {
  return new Promise((resolve, reject) => {
    if (isNaN(ms)) {
      return reject(new Error('not a valid number'));
    }
    setTimeout(() => {
      resolve(ms * 2);
    }, ms);
  });
};
```

+++

This function returns a Promise:

- Resolved after the specified milliseconds in <span style="background-color: #F2F3F4">ms</span>
    - Fulfillment value: `ms` * 2
- Rejected if `ms` ms is not a valid number
    - Rejection value: the error message

---

# Agenda

- About async/await
- Async Functions
- Handling results and errors with `await`
- Things you should know
- Some Pitfalls
- Quizzes Time

---

# About async/await

---

async/await in 7 seconds

![](assets/imgs/async-await-7seconds.gif)

（Credit: *twitter.com/manekinekko* ）

---

async/await is an ES2017 feature available in:

- Recent versions of modern browsers (Google Chrome, Firefox, ...)
- Node.js 8.x >
- Babel
- [And more](http://kangax.github.io/compat-table/es2016plus/#test-async_functions)

---

# Async Functions

---

Async function declarations: 

```js
async function foo() {}
```

Async function expressions: 

```js
const foo = async function () {};
```

Async arrow functions:

```js
const foo = async () => {};
```

Async method definitions:

```js
let obj = { 
  async foo() {}
}
```

---

Async functions always return Promises:

```js
async function asyncFunc () {
  return 123;
}

asyncFunc()
.then(x => console.log(x)); // 123
```

---

# Handling results and errors with `await`

---

## The operator `await` 

- Is only allowed inside async functions
- Waits for its operand, a Promise, to be settled

---

Is only allowed inside async functions:

```js
function nonAsyncFunc () {
  await Promise.resolve();
}
```

This is not allowed either:

```js
async function asyncFunc () {
  function nonAsyncFunc() {
    await Promise.resolve();
  }
}
```

Both will fail with:
```
Uncaught SyntaxError: Unexpected identifier
```

---

Waits for its operand, a Promise, to be settled:

- Is resolved: the fulfillment value is returned

```js
async function asyncFunc () {
  const result = await delay(100);
  console.log(result); 
  // Output: 200
}

asyncFunc();
```

+++

- Is rejected: the rejection value is thrown

```js
async function asyncFunc () {
  try {
    await delay("buh");
  } catch (err) {
    console.error(err.message); 
    // Output: "not a valid number"
  }
}

asyncFunc();
```

---

If something besides a Promise is passed to await, it just returns the value as-is:

```js
async function asyncFunc() {
  const result = await 'hola';
  console.log(result);
  // Output: hola
}
asyncFunc();
```

That means that if an array of Promises is passed, they will not be awaited:

```js
async function asyncFunc() {
  const result = await [delay(100), delay(200)];
  console.log(result);
  // Output: [Promise, Promise]
}
asyncFunc();
```


---

Handling multiple asynchronous results sequentially:

```js
async function asyncFunc() {
  const result1 = await delay(100);
  console.log(result1); // 200
  const result2 = await delay(200);
  console.log(result2); // 400
}
```
Equivalent to:

```js
function asyncFunc() {
  return delay(100)
  .then(result1 => {
    console.log(result1); // 200
    return delay(200);
  })
  .then(result2 => {
    console.log(result2); // 400
  });
}
```

+++

Handling multiple asynchronous results concurrently:

```js
async function asyncFunc() {
  const [result1, result2] = await Promise.all([
     delay(100),
     delay(200),
  ]);
  console.log(result1, result2); // 200 400
}
```

Equivalent to:


```js
function asyncFunc() {
  return Promise.all([
    delay(100),
    delay(200),
  ])
  .then([result1, result2] => {
      console.log(result1, result2); // 200 400
  });
}
```

---

Handling errors:

```js
async function asyncFunc() {
  try {
    await delay('buh');
  } catch (err) {
    console.error(err);
  }
}
```

Equivalent to:

```js
function asyncFunc() {
  return delay('buh')
  .catch(err => {
     console.error(err);
  });
}
```

---

# Things you should know

---

## Async functions are started synchronously, but are settled asynchronously

+++

1. A Promise `p` is created when an async function is executed.

2. Execution of the body starts synchronously.

    1. Execution finish via return or throw (permanently)
    2. Execution finish finish via `await` (temporarily, it will continue later on)

3. `p` is returned

+++

Execution finish via return or throw (permanently):

```js
async function asyncFunc() {
  console.log('inside asyncFunc()');
  return 'abc';
}

asyncFunc()
.then(x => console.log(`Resolved: ${x}`));

console.log('main');

// Output:
// inside asyncFunc()
// main
// Resolved: abc
```

+++

Execution finish finish via `await` (temporarily):

```js
async function asyncFunc() {
    console.log('inside asyncFunc()');
    await "dummy";
    console.log('after await')
    return 'abc';
}
asyncFunc()
.then(x => console.log(`Resolved: ${x}`));
console.log('main');

// Output:
// inside asyncFunc()
// main
// after await
// Resolved: abc
```

---

## Returning Promises

Returned Promises are not wrapped:

```js
async function asyncFunc() {
  return Promise.resolve(123);
}
asyncFunc()
.then(x => console.log(x)) // 123
```


Returning a rejected Promise leads to the result of the async function being rejected:

```js
async function asyncFunc() {
  return Promise.reject(new Error('Buh'));
}
asyncFunc().catch(err => console.error(err)); 
// Error: Problem!
```

+++

That allows us to forward fulfillments and rejections of another asynchronous computation without `await` :

```
async function asyncFunc() {
  return delay(100);
}
```

This works as well, but the code above(the one without `await`) is more efficient:

```
async function asyncFunc() {
  return await delay(100);
}
```

---

# Some Pitfalls

---

## One tip: 

Remember we are dealing with **Promises** here

---

Forgetting `await`:

```js
async function asyncFunc() {
  const value = delay();
  if (value) {
    console.log(`We have a value`);
  }
}
```

Bugs can be difficult to detect, like in the code above.

---

Performance problems:


```js
async function asyncFunc() {
  const value1 = await delay(100);
  const value2 = await delay(200);
  const value3 = await delay(300);
  console.log(`Values: ${value1}, ${value2}, ${value3}`);
}
```

(This is more of a bad code design problem though...)

---

Callback-based utilities can be tricky to use:

```js
async function getDelayResults (array) {
  return array.map(ms => {
    const result = await delay(ms);
    return result;
  });
}
```

The code above will produce a SyntaxError, the innermost function that surrounds an `await` is not async.

+++

This code works but the result is an array of Promises:

```js
async function getDelayResults (array) {
  return array.map(async ms => {
    const result = await delay(ms);
    return result;
  });
}

getDelayResults([100, 200, 300])
.then(x => console.log(x)); // [Promise, Promise, Promise]
```

+++

It can be solved using `Promise.all()`:

```js
async function getDelayResults (array) {
  const pArray = array.map(async ms => {
    const result = await delay(ms);
    return result;
  });
  const result = await Promise.all(pArray);
  return result;
}

getDelayResults([100, 200, 300])
.then(x => console.log(x)); // [200, 400, 600]
```

+++

Or using a `for of` loop:

```js
async function getDelayResults (array) {
  const result = [];
  for (ms of array) {
    result.push(await delay(ms));
  }
  return result;
}

getDelayResults([100, 200, 300])
.then(x => console.log(x)); // [200, 400, 600]
```

This runs in series though.

+++

- Depending on the case the way of solving it is different

- Solutions that run concurrently might become kind of complex

- Same problem may occur using `Array#forEach()`, `Array#reduce()`, `Array#filter()`, etc.

+++

Utilities that make iteration easy:

https://github.com/toniov/p-iteration

---

# Quizzes

---

1) What will be the output?

```js
async function ping () {
  await delay(300);
  console.log('ping');
}

async function pong () {
  await delay(100);
  console.log('pong');
}

Promise.all([ping(), pong()]);

console.log('main');
```

+++

![](assets/imgs/ok1.jpg "ok")

+++

![](assets/imgs/bad.png "bad")

---

2) What will be the output?

```js
async function ping () {
  console.log('ping');
  await delay(300);
}

async function pong () {
  await delay(100);
  console.log('pong');
}

ping();
pong();

console.log('main');
```

+++

![](assets/imgs/ok2.png "ok")

+++

![](assets/imgs/bad.png "bad")

---

3) What will be the output?

```js
async function ping () {
  console.log('ping');
  await delay(300);
}

async function pong () {
  await delay(100);
  console.log('pong');
}

ping();
pong();

console.log('main');
```

+++

![](assets/imgs/ok3.jpg "ok")

+++

![](assets/imgs/bad.png "bad")

---

4) What will be the output?

```js
async function printDelayResults (array) {
  array.forEach(async ms => {
    const result = await delay(ms);
    console.log(result);
  });
}

printDelayResults([100, 300, 200])
.then(() => {
  console.log('finish');
});

console.log('main');
```

+++

![](assets/imgs/ok4.png "ok")

+++

![](assets/imgs/bad.png "bad")

---

5) How can the performance of this function be improved?

```js
async function asyncFunc() {
  const v1 = await delay(300);
  const v2 = await delay(200);  
  const v3 = await delay(100);
  return v1 + v2 + v3;
}
```

+++

![](assets/imgs/ok1.jpg "ok")

+++

![](assets/imgs/bad.png "bad")

---

6) Something is not necessary here

```js
async function asyncFunc() {
  await delay(100);
  return await delay(200);
}
```

+++

![](assets/imgs/ok4.png "ok")

+++

![](assets/imgs/bad.png "bad")

---

## Reference

[Exploring ES2016 and ES2017](http://exploringjs.com/)
<br>
by Dr. Axel Rauschmayer

---

Thanks for your attention!

![](assets/imgs/thanks.jpg "Mio&Yukko")

Questions?
