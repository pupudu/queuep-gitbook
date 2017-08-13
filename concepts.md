# Core Concepts

There are 3 main concepts, that a developer should be aware of, to take the maximum out of QueueP. All other features of QueueP can be collectively thought of as just configurations or settings.

### 1. Publish-Consumer Coupling

The first thing to note here is that Publish is a verb and Consumer is a noun. In other words, Publish is a **function call**, and consumer is a **function definition**. Publish is executed once\(by the developer\), and executes instantly, while consumer is called multiple times as required\(by the QueueP framework\), but executes with a variable delay. The delay is added in order to distribute the load along the time axis.

The consumer, which instructs QueueP on how to process a data chunk published to a queuep queue, should be provided when initializing the queue. Then you can publish data to the queue from anywhere in your application and will be consumed by the consumer if the data chunk is identified as dirty by the dirty checker\(2nd concept of QueueP\).

```js
import qp from 'queuep';

let myQueue = qp.initQueue("my_queue", {
    consumer: function(key, data, callback) {
        // Do something with the key and data
        // Then call the callback to signal QueueP that the processing has finished
    }
});

qp.publish("app_online_status", deviceId, onlineStatus);
```

### 2. Dirty-Checker

Perhaps the heart of QueueP, the dirty checker is also a **function definition** and should be provided when initializing a queuep queue. The primary goal of the dirty checker is to decide whether or not to process a data chunk published to a queue. Dirty checker receives the newly published data and the previously published data as arguments and, based on those two, should return true or false to signal the consumer to process the newly published data.

**If you do not provide a dirty checker when initializing a queue, the inbuilt naive dirty checker will be used by default. The naive checker will basically perform a deep assertion between the old and new data. **

To demonstrate a custom dirty checker, let's say that you are publishing integers as data to the queue, and you want to process the number only if the new number is greater than the old value. You can do,

```js
let myQueue = qp.initQueue("my_queue", {
    consumer: someConsumer,
    dirtyChecker: function(oldNumber, newNumber) {

        if (newNumber > oldNumber) {
            return true;
        } else {
            return false;
        }

        // In short, return newNumber > oldNumber;
    }
});
```

While you can write the dirty checker all by yourself, QueueP provides several configurable dirty checker templates which can be quite handy to use.

For example, if you are publishing JSON objects to the queue as data and want to process the data only if some fields in the data object have changed, you can use the `makeObjectDirtyChecker` template to create a dirty checker for you.

For example,

```js
let myQueue = qp.initQueue("my_queue", {
    consumer: someConsumer,
    dirtyChecker: function(oldData, newData) {
        if (oldData.property1 !== newData.property1) {
            return true;
        }
        if (oldData.property2 !== newData.property2) {
            return true;
        }
        return false;
    }
});
```

Can be written as,

```js
import qp, {makeObjectDirtyChecker} from 'queuep';

let myQueue = qp.initQueue("my_queue", {
    consumer: someConsumer,
    dirtyChecker: makeObjectDirtyChecker("property1", "property2")
});
```

### 3. Events

The 3rd concept of QueueP is that all important information about the queuep queues are notified to the user as events. Currently queuep supports the following events

* error
* empty
* full
* duplicate
* enqueue
* consume

You can use the `on` method of a queue to register an event listener to the queue. If you need to log the error every time something goes wrong when processing a published data chunk, you can register a listener as follows.

```js
myQueue.on("error", function(err) {
    logger.error(err.message);
});
```

The `on error` event listener will receive the actual `error` as the argument. All other listeners will receive the `key` of the data chunk as the argument.

