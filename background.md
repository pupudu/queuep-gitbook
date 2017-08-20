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