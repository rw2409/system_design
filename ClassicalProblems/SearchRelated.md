#Design Webcrawler#
##Factors##
[Notes](http://www.jiuzhang.com/problem/44/)
[Crawler Implementation](http://blog.csdn.net/historyasamirror/article/details/7061059)

**In general it is a prioritized BFS like iterative process**

1.What is Internet
	* Pages are nodes
	* Links are edges

1. Fetcher/DNS Resolver
DNS Resolver: resolves the DNS
Fetcher: downloads the content
Taking time, we can use async process

1. Content Seen
Use hash function to calculate finter print in order to avoid duplicated visits.

1. URL Extractor
* Highly related to application
* extract urls from download pages

1. URL Seen
File system based key-value store, to store shortenedURL

1. URL Set
* file systems/database of pending urls from URL extractor

1. URL Frontier
  1. Start with seed URL
  1. Send Web pages from fetcher into URL extractor
  1. remove duplicated URLS and add URLs into URL Set
  
  * Be friendly to a specific website, needs a FIFO queue for each site.
   (or use table to keep track of cooldown)

1. Distributed
Hash URL into N nodes.

2. 抓取算法
  1. Each webpage is associated with weight
    1. 是否属于一个比较热门的网站 
    2. 链接长度 (?)
    3. link到该网页的网页的权重 
    4. 该网页被指向的次数 等等。


3. 网络模型
    1. One node has multiple threads
    1. Use machine from the same data center

4. 实时性
    1. Seperate news from other types
    1. Use manual config of news 
    1. Use machine learning to detect news website

5. 网页更新
	1. Track time to visit each page and adjust the TTL based on if the content has changed.

#Query data attributes#
每个record有个很大field，比如年龄，性别，爱好等。给一个field的组合，比如小于25岁，爱好体育, query满足这些组合条件的用户个数 
* Use lucene to build index.
* Shard the data into multiple partitions.

#Real Time Search#
[Linkedin Zoie](http://www.cnblogs.com/forfuture1978/archive/2010/11/29/1891476.html)
* Write incoming index into RamA, with existing Disk Index
* When RamA reaches to certain size, Ramup up RamB and write incoming index into RamB, change RamA to be read-only and start copying into Disk
* Copy finished, Re-open Index Reader on Disk

#News Feed Item Rank#
#Edge Rank#
∑ – The sum of each individual edge. An edge is a story that can show up in your News Feed, like a status update, comment, Like, tag, and so on.
u – The affinity score. This is the factor that weighs how close you are with the person doing the posting. If you frequently interact with the person posting, have several mutual friends, or are related, Facebook is more likely to give that content a higher weight.
w – The weight for this edge. Not all actions are considered equal in the eyes of Facebook’s algorithm. For example, a friend creating a status update would carry more weight than someone simply liking a status update.
d – The time decay factor. As a posts gets older, it’s more likely that it has already been seen or that it is no longer as relevant. Facebook remedies both of these problems by taking the age of the post into consideration.

#ML based Rank#
Building a probalistic model based on how user interacts with previous posts. Rank based on the probability.
