#Entities
##RDB
  * user_profile
  * tags
  * media metadata

#Media Store
  * use S3 for raw data/downsamped data
  * Amazon CloudFront as CDN

#Feed
##Relational database design
posts
  * post_id
  * author
  * contentkey
  * timestamp

following
  * source_id
  * dest_id

Generate feeds for user 'foo' with last 100 messages:

select *
from posts
where author in (
	select dest_id
	from following
	where source_id = 'foo'
)
order by timestamp desc
limit 100
Simple for write but super expensive for read.

## Fanout-on-write(Push) Model using Key-Value
Feed:
 1. Feeds
    * UserId,
    * List<Media> (Redis)
 1. Feeds (DynamoDB like)
   * UserId (ParitionKey)
   * MediaId (SortKey)

Timeline:
 similar table structure as Feeds, but for own stream instead of following.

Media:
 * mediaId
 * timestamp
 * content

With inbox model (Good with high R:W ratio 100:1)
For each post, delivered to each receipants inbox.
  * sorted list of user feeds
  * O(1) read
  * O(N) write N=followers

1. For each post, breaks in two steps
  1. Start a workflow (persistanted using db record) (SWF,RabbitMQ) and acknowledge to client.
  2. The workflow/task dispatch system will kick off child activities:
	* Fan-out to followers
	* append to timeline
	* Spam Analysis
	* Search indexing
	* Cross-Network posting
1. For dynamoDB kind data-structure
  * No need to build extra on sychronization/item potent
1. For Redis like persist all messages in a row
  * Synchronization: multiple writer trying to update the same inbox, optimistic locking for list model using version/timestamp
  * Delivery times:  retried workflow may enqueue messages twice.

Drawbacks:
  * Ordering/Relevance could be difficult.

## Pull model
1. Pulling servicing node has huge load
1. Need good cache
1. Order by reverse timestamp, for recent posts stored in queue/cache, for order backed up in disk (MySQl).

##Pintrest Smart Feed##
[SmartFeed](http://itindex.net/detail/54760-pinterest-feed-%E6%9E%B6%E6%9E%84)
* SmartFeedWorker: workflow of a post feed, offer to priorityQueue to each feed follower, together with scoring metadata
* SmartFeedContentGenerator: 
  * Pull all latest pin after last read. 
  * Can re-order/shuffle based on sources to diversify but cannot change the priority.
  * Can delete from priorityQueue
* SmartFeedService: 
  * Generate Feed view
  * Pulling latest view from FeedContentGenerator
  * Materialize user feed after sucessful poll and call FeedContentGenerator to delete
  * Persist FeedItem into storage (latest in memory, old in disk)
  * When timeout from smartFeedContentGenerator, just always load old ones.
* Double data store (all uses replicated HBase)
  * Worker/ContentGenerator: PriorityQueues. 
  * FeedService: Materialized content.
* Each write needs a replication to a slave backup

#Optimize news photo feed for mobile#
* Photo/Multimedia CDN
* Local snapshot of content and paginated
* Reduce polling and use push notification
* Protocol for image downloading
* Data Response format more efficient than Json
