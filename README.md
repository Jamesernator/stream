
# AsyncQueue

### Introduction

Soon JavaScript will be getting [asynchronous iterators and asychronous generators](https://github.com/tc39/proposal-async-iteration), however the proposal doesn't include any way for creating an asynchronous sequence from existing asynchronous operations.

This library intends to fill that void by providing a single concrete type which has large similarties to the Promise type, it also borrows ideas from the [obervable proposal](https://github.com/tc39/proposal-observable) in its minimal API design and use of an observer object.

### API

#### The AsyncQueue constructor

The `AsyncQueue` constructor is analogous to the `Promise` constructor. It takes as its first argument an initializer function which is passed an object we'll refer to as the Queue Signaler object. Its optional second argument is simply an object that can specify the following properties.

The first property is `maxSize`, this is the maximum number of elements that the queue can contain before it stops dropping newly arrived values by default this is `Infinity`.

The second property `drop` may be either value `"newest"` or `"oldest"` and determines which end of the queue we'll drop values from when the `maxSize` is exceeded, when this value is `"oldest"` (the default) it will drop the oldest values from the queue to keep the size below `maxSize`, otherwise when the value is `"newest"` it will drop the newest values.

The initializer function should also return a function that is used up for cleanup operations if the queue is closed early. This cleanup function will be called when either the queue is signaled to finish or throw an error or when the queue is closed prematurely via `asyncQueue.return`.

```js
const clicks = new AsyncQueue(queue => {
    // The initializer function may use the Queue Signaler to send values
    element.addEventListener('click', queue.yield)
    return _ => { element.removeEventListener(queue.yield) }
}, {
    maxSize: 1, // Only store the most recent click
                // newer clicks that aren't handled are simply dropped
    drop: 'newest', // Only the oldest click will be kept in the queue until
                    // it is consumed
})
```

### The Queue Signaler
