# Background

I wrote queuep to fix an issue in a project I was working on. 
I believe, summarizing that story will give you a sound understanding about what exactly queuep does. 
So here it goes,

There is an API endpoint in the application to which thousands of devices send periodic data. 
The data is about the online status of an application running on each device. 
So we were interested only in the transitions. However, the online status was being updated in a database 
without caring about the value. For example, If a device sends 1000 requests saying the online status is true, 
we do 1000 mysql operations to override the same value. And at some point in time, 
the database could not process any more queries and both the database and the NodeJs API stopped 
responding to further requests.

So I wrote a middleware which would stay somewhere between the API endpoint and the data access layer, 
preventing redundant mysql operations. Although the primary requirement was to prevent redundant operations, 
it gave few other benefits as well. 
Thus I was motivated to implement the module as a lightweight and easily pluggable library. 
Listed below are some of the said benefits in brief.


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
