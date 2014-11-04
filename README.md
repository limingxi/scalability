#A brief conclusion of high scalability techniques.
The content of this file will cover all the high scalability techniques I learn so far.  
I would like to classify the whole architecture into 3 layers: **load balancing layer**, **application layer** and **database layer**. At the end there will be some words about caching which I think is very useful and can be used in any layer :)

##Load Balancing Layer
The [load balancing ](http://en.wikipedia.org/wiki/Load_balancing_%28computing%29) layer can be treated as a front tier. Every request coming in will be handled by load balancing layer first. And the reason we want to use load balancing is that we want to distribute the requests evenly on all the servers which will make servers response fast. As you may know that www.google.com does not have only one IP address. So when you are sending a request to use google service, the IP address you get from DNS server can be different from the other users. The load is balanced from the very beginning by DNS server. The algorithm for balancing can be [round-robin](http://en.wikipedia.org/wiki/Round-robin_DNS) or hashing the IP address of the requests.  

There is also a very important techniques here called [content delivery network ](http://en.wikipedia.org/wiki/Content_delivery_network). Because the network transmitting latency is always there, CDN solves this by distribute data center to different locations.  

The main problem of load balancing is the persistence within one session. If user 1 is at first served by server 1 and then the second request is balanced to server 2, the user should not feel any difference. In this case, all the application servers should have the same code base and the session data should be stored in another server which is visible by all application servers. Some knowledge here is concluded from a wonderful [ post ](http://www.lecloud.net/post/7295452622/scalability-for-dummies-part-1-clones).  

##Application Layer
OK, now the balancer scheduled very well and the requests have been sent to the application server. As we know, the server's computing resource is also limited by hardware. With the requests coming in(especially TCP based), we have two options to handle:   

1. the first is just handle them **one by one** (synchronously) which will make the response so slow and your users may just shutdown the application  
2. the other option is to use **multi-thread** (asynchronously) strategy, we open one thread for each client.  

In fact [Apache](http://en.wikipedia.org/wiki/Apache_HTTP_Server) is using the second strategy (maybe multi-process, but we just use multi-thread for general) and it can serve very well in most cases. Although open and destroy threads can be time consuming, [thread pool](http://en.wikipedia.org/wiki/Thread_pool_pattern) can solve this problem. However, multi-thread can exhaust the server's computing resource. There is a famous question related to this topic: [C10K problem](http://en.wikipedia.org/wiki/C10k_problem).  

You may have heard that [Nginx](http://en.wikipedia.org/wiki/Nginx) is the solution of C10K problem. But why is that? If you have experience in socket programming in C, you know there are two ways to make application work asynchronously: multi-thread or use the function [select](http://linux.die.net/man/2/select).   

The select function acting like this: we have lots of sockets here and we put them into a set. Every time we start the loop, the select function is called and it will traverse the socket set and ask: you ready for read/write? Once it gets a positive response, it will handle that socket's I/O. This mechanism is called **polling**. As you can see, CPU does not wait for any single socket's I/O, it only handle the sockets once they are ready.  

Polling is good, but there is another mechanism which is more efficient: **event driven**. Other than asking the sockets one by one, event driven mechanism will give a trigger to each event(like disk I/O or network I/O). Once I/O is finished, the event will pull the trigger and say: Sorry to interrupt, but I'm ready now, please serve me first. This is clearly more efficient than polling and that's the secret of Nginx.  

By the way, [**nodejs**](http://blog.mixu.net/2011/02/01/understanding-the-node-js-event-loop/) is also a very popular term in the world of event driven. It is very fast and single thread. That would be thankful for javascript supports **callback**(Which acts like the trigger mentioned above) and **google's V8 engine**.  

##Database Layer

Now we come to the most headache part: database. Disk I/O is the slowest and that may be your application's bottleneck.  

One solution should be: **master-slave schema**. The core idea of master-slave is to separate read and write. The master server is in charge of updating and slaves are just used for reading. The content of master database will be replicated to slaves periodically. Clearly this is not a real time updating, it can work fine on static data. What about real time changing data?  

Another solution is combine your relational database with **[nosql](http://en.wikipedia.org/wiki/NoSQL)** database. Nosql is short for Not Only SQL, it contains a lot of products, everyone has its own specific field. The point of nosql is: it can be [horizontally scaled](http://en.wikipedia.org/wiki/Scalability) easily! Horizontally scale is important, but it is a disaster for sql database. Since sql database has several normal forms and supports things like transaction, referential integrity etc, it is hard to handle the situation like make tables on different servers have some "relation". However, nosql only needs key to retrieve value, the "relation" is gone. So nosql database is easy to scale horizontally.  

