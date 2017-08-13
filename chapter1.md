### Usage Patterns

To make queuep flexible and blend with any type of code base, we have allowed different ways of interacting with the library. In other words, queuep usage will not look like some alien code in your code base.

In this section we will summarize few common ways/variations of interacting with queuep.

### 1. Interaction

When you initialize a queue with a preferred Id, QueueP will return a reference to the queue instance.

```js
import qp from 'queuep';

let myQueue = qp.initQueue("my_queue", {
    consumer: someConsumerFunction
});
```

This reference can be used to perform actions with the initialized queue. For example to publish some data to the queue,

```js
myQueue.publish("my_queue", key, data);
```

QueueP also keeps track of the Id, so that you can get a reference to the queue from anywhere in your application. For example from a different module you can do the following to get statistics about the queue,

```js
import qp from 'queuep';

let myQueue = qp.getQueueInstance("my_queue")

myQueueStats = myQueue.getStats();
```

It also allows you to interact with a queue directly via the class by providing the Id. For example, to do the same as above you can do,

```js
import qp from 'queuep';

let myQueueStats = qp.getStats("my_queue")
```

### 2. Promises and Callbacks

QueueP is designed to support both callbacks and promises by default. All asynchronous methods in QueueP comply with this standard. For example when publishing data to a queue and you have to do something after the operation,

```js
myQueue.publish("my_queue", key, data, () => {
    // Do something in this callback 
})
```

or you can use the promise returned\(Do not provide the callback argument here\)

```js
myQueue
    .publish("my_queue", key, data)
    .then(() => {
        // Do something in this callback 
    })
```

**All Callbacks in QueueP are error first callbacks, **meaning that the first argument will always be an error if any, and the proceeding arguments are optional data arguments.

The consumer function definition provided when initializing the queue can also be in both these forms. It is worth noticing how this is different from the rest. Here the QueueP user should return the promise, or call the callback to signal queuep that the asynchronous consumer function has finished executing. If it helps, this is similar to how the promise and callback pattern works in the mocha test framework.

Promise based usage:

```js
let myQueue = qp.initQueue("my_queue", {
    consumer: (key, data) => {
        return new Promise((resolve, reject) => {
            // Do some asyn task here and call reject or resolve to 
            // signal QueueP that the operation has finished executing
        })
    }
});
```

Callback based usage:

```js
let myQueue = qp.initQueue("my_queue", {
    consumer: (key, data, callback) => {
        // Do some asyn task here and call the callback to signal QueueP
        // Callback is in error-first format
        // If an error occurs, call: return callback(new Error("Some error"));, else call: return callback();
    }
});
```

### 3. Synchronous Functions

It is worth mentioning that some functions that queuep exposes are synchronous. That means, these functions return the desired output value directly without a promise or accepting a callback. Some examples are,

* `getStats()`
* `printStats()`
* `on()`
* `getQueueInstance()`

This is because these function implementations are trivial and we thought that an asynchronous interface would be just a burden for the QueueP users.

---



