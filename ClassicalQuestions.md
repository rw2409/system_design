#Online behavior
* Design amazon's last viewed product page (can goes back to last 100)
* Design amazon item recommendation system.
* Design Airbnb recommendation system.

#News Feeds/Social Network
* Design a [picture sharing website](http://highscalability.com/blog/2011/12/6/instagram-architecture-14-million-users-terabytes-of-photos.html). How will you store thumbnails, photos? Usage of CDNS? caching at various layers etc.
* Design a news feed (eg facebook , twitter): [news feed](http://www.quora.com/Software-Engineering-Best-Practices/What-are-best-practices-for-building-something-like-a-News-Feed)

#Search
* [Search engine](http://infolab.stanford.edu/~backrub/google.html) (generally asked with people who have some domain knowledge ) : basic crawling, collection, hashing etc. Dependes on your expertise on this topic
* Design a site like [junglee.com](http://www.junglee.com/) i.e price comparision , availability on ecommerce websites. When and will you cache, how much to query, how to crawl efficiently over ecommerce sites, sharding of dbs, basic db design
##GeoSearch
* Design a product based on maps, eg hotel / ATM finder given a location. 

#Stream processing, Counting, Trending
* (very common:) top 'n' or most frequent items of a running stream of data
  * [Solution](https://github.com/rw2409/system_design/blob/master/ClassicalProblems/TopOrMaxFromStream.md)
* Design a logging system
  * Design a fraud/security risk behavior detection system
  * [Solution](https://github.com/rw2409/system_design/blob/master/ClassicalProblems/LoggingMetricsSystem.md)
* Design election commission architecture :
  * [Solution](https://github.com/rw2409/system_design/blob/master/ClassicalProblems/ElectionCommission.md)
* [High Performance Counters](https://github.com/rw2409/system_design/blob/master/ClassicalProblems/HighPerformanceCounters.md)
* [Counting related problems](https://github.com/rw2409/system_design/blob/master/ClassicalProblems/CountingRelatedProblems.md)  
  * Website Visitor Count
  * Trending HashTags
  * Photo referencing/comments count
  * Music play
* [Probalistic Data Structures](https://github.com/rw2409/system_design/blob/master/ClassicalProblems/ProbalisticDataStructures.md) 

Delivery times (exactly once), scaling, delay, expected accuracy.

#Simply Store
* Design a [url compression system] (http://www.hiredintech.com/system-design/the-system-design-process/)
* Design dropbox's architecture. [good talk on this](https://www.youtube.com/watch?v=PE4gwstWhmc)
* Design a system for collaberating over a document simulataneously (eg [google docs](https://neil.fraser.name/writing/sync/))
* A web application for chatting, eg [whatsapp](http://highscalability.com/blog/2014/2/26/the-whatsapp-architecture-facebook-bought-for-19-billion.html), facebook chat. Issues of each, scaling problems, status and availablility notification etc.

#Online interaction Chatting
* Design an online poker game for multiplayer. Solve for persistence, concurrency, scale . Draw the ER diagram for this 

#Geo POI
* [POI nearby Solution](https://github.com/rw2409/system_design/blob/master/ClassicalProblems/Geo-Poi.md)

#Low-level
* Design malloc, free and [garbage collection system](http://courses.cs.washington.edu/courses/csep521/07wi/prj/rick.pdf) . What data structures to use? decorator pattern over malloc etc.
