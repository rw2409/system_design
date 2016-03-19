Optimistic locking will not help, too many writes need to happen
Writing to disk is slow and locking will take too long.

#Use cases#
* votes count
* Website visitor count
* Number of comments

#Sharding Counters#
* Set-up
  * N paritions, key value store
  * Trying to avoid serialized writes into the same entity
* Process
  * insert: find a random counter out of N shards, increment it
  * read: sum counter from all paritions.
* Pros/Cons:
  * reduces the write contention to 1/N.
  * A little bit Expensive
  * Write is slow (disk seek time)
#Cached Read Counters#
* Set-up
  * Single Memcached
  * Persistent DB
* Process:
  * insert:
    1. ThreadSafe update the value in memcache
    2. If the returned updated value is evenly divisible by N
    3. add N to the datastore counter
    4. decrement memcache by N
  * read:
    1. Read from DB (could be cached)
* Pros/Cons:
  * Count is delayed, no need to be 100% accurate
  * Single Point of Failure
  * Write is fast

#Combination of the above #
* Set-up
  * Multiple Memcached
  * Persistent DB
* Process:
  * insert:
    1. Pick a random memcache shard
    1. ThreadSafe update the value in the shard
    2. If the returned updated value is evenly divisible by N
    3. add N to the datastore counter
    4. decrement memcache by N
  * read:
    1. Read from DB (could be cached)
* Pros/Cons:
  * Count is delayed, no need to be 100% accurate
  * No Single Point of Failure
  * Write is fast
  * Contention is small
