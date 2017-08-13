# QueueP

Pronounced: "queue-pea"

QueueP is a framework originally designed for congestion control in NodeJs applications. With time, it was improved to be able to run on **any **JavaScript application. It is more often useful for scenarios where redundant requests should be ignored. However, QueueP can be used in any other case where performance is affected by a heavy load of data.

### Why QueueP?

None of the similar queue libraries have a concept of avoiding duplicate requests \(At least from the ones I found\). QueueP filters out redundant requests and allows the API to process useful data without requiring you to upgrade physical resources unnecessarily.

QueueP also allows you to customize the logic of deciding whether or not to process a data chunk via the concept of dirty checkers. While you can write the dirty checker all by yourself, QueueP provides several configurable dirty checker templates which can be quite handy to use.

### Installation

To install the stable version:

```bash
npm install --save queuep
```

QueueP is in ES5 syntax meaning that you can directly use it in your webapps. If using NodeJs, you will need NodeJs v4 or later.

### Basic Usage

Let's assume that 5000 devices are sending data to an endpoint about their online status.

First, initialize a queue\(can have multiple queues\) with the minimal required configurations. Constructor of a module is a good place to keep the initQueue code.

```js
import qp from 'queuep';

qp.initQueue("app_online_status", {
    consumer: updateOnlineStatus
});
```

Then you can publish data to the queue.

```js
qp.publish("app_online_status", deviceId, onlineStatus);
```

initQueue and publish method calls can be in the same module or in different modules. I personally prefer to keep the initQueue and publish methods in the same module. But that is completely up to you to decide.

* Note 1: The argument**id**is used to identify the queue. An application can have any number of queuep queues.

* Note 2: The argument**consumer**should be a function reference which should be either;

A function which accepts 3 arguments. First two arguments will give the key and the data published from the publish method. The 3rd argument is an error-first callback that should be called to signal QueueP that the consume task has finished. See example:

```js
let updateOnlineStatus = (key, data, callback) => {

    let onlineStatus = data,
        deviceId = key;

    deviceDao.updateOnlineStatus(deviceId, onlineStatus, function (err) {
        if (err) {
            return callback(err)
        }
        return callback();
    });
}
```

* A function which accepts 2 arguments and returns a promise. First 2 arguments are as same as above. The promise can be resolved or rejected to signal QueueP that the consume task has finished. 

See example:

```js
let updateOnlineStatus = (key, data) => {

    let onlineStatus = data,
        deviceId = key;

    return new Promise((resolve, reject) => {
        deviceDao.updateOnlineStatus(deviceId, onlineStatus)
            .then(resolve)
            .catch(reject);
    });
}
```

### Background {#license}

I wrote queuep to fix an issue in a project I was working on. I believe, summarizing that story will give you a sound understanding about what exactly queuep does. So here it goes,

There is an API endpoint in the application to which thousands of devices send periodic data. The data is about the online status of an application running on each device. So we were interested only in the transitions. However, the online status was being updated in a database without caring about the value. For example, If a device sends 1000 requests saying the online status is true, we do 1000 mysql operations to override the same value. And at some point in time, the database could not process any more queries and both the database and the NodeJs API stopped responding to further requests.

So I wrote a middleware which would stay somewhere between the API endpoint and the data access layer, preventing redundant mysql operations. Although the primary requirement was to prevent redundant operations, it gave few other benefits as well. Thus I was motivated to implement the module as a lightweight and easily pluggable library. Listed below are some of the said benefits in brief.

###### Normalize Traffic Spikes

Due to the way queuep works internally, sudden spikes of data traffic will be normalized. That is, the load will be distributed along the time axis and hence CPU usage of the server will be optimized with no extra effort.

&lt;Diagram coming soon&gt;

###### Bounded Resource Utilization

When the load received is too damn high, and ignoring duplicates and distributing along the time axis also cannot help, queuep will sacrifice unprocessed old data to keep the application alive. \(For example, if the \(n+1\)th request received before processing the \(n\)th request, the \(n+1\)th data will replace the \(n\)th data\).

###### No Starvation

The internal algorithm used in QueueP takes measures to avoid starvation without compromising performance. Algorithm is based on the FIFO principle.

###### Backed By Redis

QueueP supports\(optional\) redis as the intermediate storage for published data. This ensures that queued data is preserved even if the NodeJs application was restarted.

Earlier versions of QueueP had the redis strategy built in. However, redis strategy was taken out and was implemented as a plugin to queuep. You can install it with,

```bash
npm install queuep-redis
```

Please view the short and sweet documentation at [https://www.npmjs.com/package/queuep-redis](https://www.npmjs.com/package/queuep-redis)

This will make the core library even lighter while opening the opportunity to use other methods of intermediate storage such as rabbitMq or nats.

### License {#license}

MIT

---

