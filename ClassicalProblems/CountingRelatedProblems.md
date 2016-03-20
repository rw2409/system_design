#Website Visitor Count#
A website/service has big number of visitors, build a VisitRecordSystem that
- get_hits_last_5mins()
- record_hit()
//record_hit() should be a async process.(not sync call)
##Very hight volume##
* Multiple Thread: update in each shard of counter should be serialized (Atomic or synchronized).
* Distributed: client/counters are distributed, but final data should be aggregated in a single database.

##Solution##
* Counter implementation, we can use the high-performance counter options.
  * If no delay is allowed, we should have each write directly to DB (shareded) and each read will need to group all related shards.
    * We can use the time minute/seconds index from the day 0 and use it as part of the key.
    * This set up will need multiple DB to share write load.
  * If delay is allowed, we can implement write buffer in memory and write to DB finally to group. This will reduce the DB load.

#Photo referencing/comments count#
Design photo reference counting system at fb scale 在每个Appserver 上对每个photo加一个counter,然后每隔T时间传到一个aggregator 把所有与目标相关的counter相加，然后update DB和Memcached. 一些细节还没想清楚。
**MAYBE TAO is better?**
* This value is very small compared to site visits, but the user is more cautious about this data.
* We can update in a cache initially and then increment after certain period.
  * Needs a back-up of Master/Slave or LocalSnapshot File
  * Periodically write to DB
* If below 100 TPS/second, we should be able to use distributed key-value store (not sharded) as counter.

#Most shared URLs#
Rank the most shared urls for the last 10 minutes, for last hour, for last day, etc.
There are total 100 millions url sharing happen every day.
* Each time a URL is shared, an asycn process should be published to recordShare(URL)
* Batch job can work, but it has scaling problem.
* We can introduce a shared

#Trending Detection#(Instagram)
##Definations##
* Trend: Something mentioned more than usual (poplarity, novelty, timeliness)
* Something: search terms, hashTags, placeTag

##Identifying Trend##
1. Count Shared quantity for each tag
  * 5min window count for each tag for last week
1. C(h, t) is the counter for hashtag h at time t
  * # of posts that were tagged with this hashtag from time t-5min to time t
1. P(h, t) probability of observing the hashtag h at time t.
  * 'Normalize' from C(h,t) to P(h,t)
  * We can also use historical data of C(h,t) to build expected probability P'(h,t)
1. Trendiness
  * S(h, t) # P(h, t) * ln(P(h, t)/P’(h, t)) (K,L divergence)

##Predicting P'(h,t)##
1. Many choices
1. Cheap but work well:
  * Maximal probability over the past week
    * cheap, aggressive, can quickly identify
  * For calculating P'(h,t), five minute interval is not needed, hourly is fine (reduce noise)
  * If a word does not have big enough data, simply uses a default of 3/hour to save space to hold specific counter.

##Ranking and Blending##
rank by S(h, t) value but the value drop very quickly
* Maximal KL score for each trend, say SM(h)
* Sd(h, t) # SM(h) * (1/2)^((t - tmax)/half-life), we define the tmax
* Trend stays for hours
##Grouping##
A couple of tags talking about the same thing
* Typo or stemming
* Coourances: when usually happens together
* Topic Distribution

##System Design ##
1. Pipeline model, each stage has multiple paritions
  1. MediaStream
  1. Preprocessor:
  1. Parser: extact hashtags, places, apply filters
  1. Scorer: stores time-window counters for each trend, compute S(h,t), periodically emit S(h,t)
  1. Ranker: aggregates and ranks
  1. Ranker output is periodically grouped and stored in DB.
1. Server layer can be cached and read-through from DB.

#Music Stats#
1. system design – design facebook music system，只需要design service tie.
- get_top_10_list_music_ids(int64 userid) for last week
  * Data should be cached, Refresh every 5 min from Table UserTopPlayed
- record(int64 userid, int64 musicid, int64 timestamp)
1. Async send event {time, userId, musicId}, Background processing
  * Add UserActivities entry for the music
  * Check if Timestamp difference is longer than refresh period.
  * If longer, calculate the top played Music and save to UserTop Played
  * Archive UserActivities.
1. Async deletes all obsolete data.

###Tables###
UserActivities
-DaySeqNumber+User
-Map<MusicId, Count>

UserTopPlayed
-TimeStamp
-User
-List<MusicId>
