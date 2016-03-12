# SYSTEM DESIGN PREPARATION
* How to prepare and answer system design questions:

## Objective
I collected and studied from a lot of links while preparing for interviews this year and realized that unlike coding questions which has plenty of good repos and combined resources, system design remains elusive. People end up reading from scattered resources and might get pigeon-holed into studying one specific domain and get tongue tied when answering such questions. Hence I collected these links and design techniques for interviews, thought I should share with everyone.
If you are already familiar with the basics (given below) it will take you ~2 months to gain a strong foothold over such questions. If you have much less time, scroll down to the bottom for the tl;dr version.

## Index
- [ ] [Where to start from?](#start)
- [ ] [Basics](#basics)
- [ ] [Steps how I approach the system design questions in interviews](#myapproach)
- [ ] [Common Design questions](https://github.com/rw2409/system_design/blob/master/ClassicalQuestions.md)
- [ ] [architecture](#architecture)
- [ ] [company engineering blog links](#blog)

## <a name='start'>Where to start from?</a>

For a very broad overview please go through these lectures, really useful:
* [david malans cs75 scalability talk](https://www.youtube.com/watch?v=-W9F__D3oY4&list=PLmhRNZyYVpDmLpaVQm3mK5PY5KB_4hLjE&index=10)
Feel free to go through other lectures if needed. 

* [david huffman's talk , scaling up talk](https://www.udacity.com/course/web-development--cs253) ([Youtube link](https://www.youtube.com/watch?v=pjNTgULVVf4&list=PLVi1LmRuKQ0NINQfjKLVen7J2lZFL35wP&index=1))

* [scalability for dummies](http://www.lecloud.net/tagged/scalability)

These talks should give you decent ammo to start formulating some architectures yourself. 

## <a name 'basics'>Basics</a>
### Basic concepts and knowledge to start with.
1. Operating system basics
  * file system
  * virtual memory
  * copy-on-write
  * memory swap
  * paging
  * instruction execution

2. Networking basics
  * TCP/IP stack
  * HTTP
  * DNS
  * cs75 on youtube (1st lecture) should give a broad overview

3. Concurrency basics
  * Threads/Processes
  * Locks/Mutex/Semaphore differences
  * How to use multi-thread in Java? (Executor/Thread/Runnable/Callable)
  * Future<>/Volatile?
  
4. Storage basics
  * sql vs no sql differences
  * typical sql solutions
  * typical nosql solutions
  * DB indexing/sharding
  * DB duplication of master-slave/peers
  * Caching

5. Web Service basics:
  * Restful/SOAP(RPC)
  * load balancers,
  * Cookies
  * Precompute/Caching/[CDN](http://www.51know.info/system_performance/cdn/cdn.html)
  * Security & authetication

6. Distributed architecturing
  * [CAP therem](http://robertgreiner.com/2014/08/cap-theorem-revisited/)
  * Consistant hashing
  * Zoo keeping?

7. Async processing
  * Messaging common solutions such as Kinesis/Kafka/SQS
  * Messaging topology such as Storm
  * Async workflow such as SWF

### Common Best practices to follow

1. Avoid Single point of failure: using redundancy etc
  * hosts in 3+ multiple data centers to escape from data center outage
2. Using messaging/Stream for following cases
  1. Drop synchronous dependency on non-critical process: using async
    * E.G.1: User registration on emailing/cellphone validation
    * E.G.2: RoutingDocumentRecommendation publishing
    * E.G.3: Ordering system sends event/document for order placed instead of directly calling inventory management services
    * Improve **throughput** and decouple dependency
    * Decoupling systems, better **availability**, can order even inventory system is down.
  
  2. Peak/Burst requests
    * E.G.1: RedPocket, Promotion etc which input request rate is much bigger than downstream can handle.
    * Improve throughput and availability

  3. Logging/Metrics data processing
    * E.G.: Pmet system (MetricsAgent + Pmet + Igraph/Monitor)

  4. Communication system (basic, small scale)
    * p2p or group-chat
    * Each session/chat is a message queue.
    * Queue could be pre-created and then allocated while start conversation.
3. Optimistic locking for lock-free
  * Using version number in DynamoDB
4. Build itempotent write behavior.
  * Using requestID such as FCInventoryService
  * Using lastProcessedTimeStamp or sequence number to ignore delayed events.


## <a name='myapproach'>Steps how I approach the system design questions in interviews</a>
These are the steps I go through in solving design problems and overall design practices.

1. **Understand the problem and clarifying requirements**
Never assume anything, should always clarify with business/interviewer
  * Ask for **use-cases** when not understand the system, try to clarify a use-case how the system is used for.
  * Aks for **limitations**: such as RPS, Throughput, security, cost, dependency, resource etc. 

2. Solve the problem for a **small** scale of users/items/use-cases. This will broadly help you figure out the high level **end-to-end components**.
  * Write down the various components and how each component **interact** with each other.

3. Adjust the solution to solve the problem at **full scale**. Typically this involves
  * **De-coupling** interactions between components (async, messaging etc)
  * **Extract** a new layer/component from a existing component as a new abstraction layer
  * Both of the above makes it easier to scale for each different layer using different approach.
  * Try to name concret solutions (AWS, OpenSource etc) and not to re-invent the wheel.
  * Certain component may require special best practice to build it in scalable/extensible ways.
  * Last two points above requires knowledge and experience.

## <a name='architecture'>Architectures :</a>

Personally I looked into the following architectures:
* [Basics of google search](http://infolab.stanford.edu/~backrub/google.html)
* Basics of messaging frameworks like Kafka , queuing architectures like rabbitmq.
* Broad overview and advantages of Redis , mongodb , cassandra. 
* [Google file system](http://static.googleusercontent.com/media/research.google.com/en//archive/gfs-sosp2003.pdf)
* [Google architecture] (http://highscalability.com/google-architecture)
* [Instagram](http://instagram-engineering.tumblr.com/post/13649370142/what-powers-instagram-hundreds-of-instances) and other image based social networks
* [Memcache scaling by facebook](https://cs.uwaterloo.ca/~brecht/courses/854-Emerging-2014/readings/key-value/fb-memcached-nsdi-2013.pdf)
* [Twitter scaling](https://www.youtube.com/watch?v=z8LU0Cj6BOU) and facebook feeds
* [facebook graph api](https://cs.uwaterloo.ca/~brecht/courses/854-Emerging-2014/readings/data-store/tao-facebook-distributed-datastore-atc-2013.pdf)
* [facebook haystack needle architecture](https://www.usenix.org/legacy/event/osdi10/tech/full_papers/Beaver.pdf)
* [youtube architecture and optimizations for video](https://www.youtube.com/watch?v=ZW5_eEKEC28)

## <a name='blog'>company engineering blog links </a>

Depending on where you are interviewing, go through the company blog . VERY USEFUL IN INTERVIEWS! It really helps if you have an idea of the architecture, as the questions asked will generally be of that domain and your prior knowledge will help out here.

* [Airbnb Engineering](http://nerds.airbnb.com/)
* [Bandcamp Tech](http://bandcamptech.wordpress.com/)
* [BankSimple Simple Blog](https://www.simple.com/engineering/)
* [Bitly Engineering Blog](http://word.bitly.com/)
* [Cloudera Developer Blog](http://blog.cloudera.com/blog/)
* [Dropbox Tech Blog](https://tech.dropbox.com/)
* [Engineering at Quora](http://engineering.quora.com/)
* [Etsy Code as Craft](http://codeascraft.com/)
* [Facebook Engineering](https://www.facebook.com/Engineering)
* [Flickr Code](http://code.flickr.net/)
* [Foursquare Engineering Blog](http://engineering.foursquare.com/)
* [Google Research Blog](http://googleresearch.blogspot.com/)
* [Groupn Engineering Blog](https://engineering.groupon.com/)
* [High Scalability](http://highscalability.com/)
* [Instagram Engineering](http://instagram-engineering.tumblr.com/)
* [LinkedIn Engineering](http://engineering.linkedin.com/blog)
* [Oyster Tech Blog](http://tech.oyster.com/)
* [Pinterest Engineering Blog](http://engineering.pinterest.com/)
* [Songkick Technology Blog](http://devblog.songkick.com/)
* [SoundCloud Backstage Blog](https://developers.soundcloud.com/blog/)
* [Square The Corner](http://corner.squareup.com/)
* [THE REDDIT BLOG](http://www.redditblog.com/)
* [The GitHub Blog](https://github.com/blog/category/engineering)
* [The Netflix Tech Blog](http://techblog.netflix.com/)
* [Twilio Engineering Blog](http://www.twilio.com/engineering)
* [Twitter Engineering](https://engineering.twitter.com/)
* [WebEngage Engineering Blog](http://engineering.webengage.com/)
* [Yammer Engineering](http://eng.yammer.com/blog/)
* [Yelp Engineering Blog](http://engineeringblog.yelp.com/)

* Go through cs76 and udacity's links given above for scaling systems. 
* See this talk: http://www.hiredintech.com/system-design/the-system-design-process/ and develop a process for how to answer such questions .
* I found [hiredintech](http://www.hiredintech.com/system-design) videos an excellent place to start with. The way how to approach a design question as given in the link is really useful. It goes into how we start with clearing the use-cases of the system, then thinking in abstract manner of the various component and the interactions. Think about the bottlenecks of the system and what is more critical for your system (eg latency vs reliability vs uptime etc) Address those giving the tradeoff of your appraoch. 
