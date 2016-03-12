# Problem
Given Coordinates, find POIs near-by.
* How near-by?
* How many POIs we have?
* What is SLA?
* How many (concurrent) users?

# Calcuating distance
* real-accurate distance is a sphere distance not linear distance
* for short distance, it's OK to use some estimation
* we should be able to determine rectagular area of given (lat, lng) and distance.

# Searching for near-by points
## B-tree index on lat/lng colum of pois;
poi | lat (indexed) | lng (indexed)

1. use query select * from pois where lat between MIN_LAT and MAX_LAT and lng between MIN_LNG and MAX_LNG.
2. filter results by calculating distance for each returned result and sort.
  * Select K problem, we can use heap/selection algorithm
  * Or use heap
  * there are some estimations can be used for distance calculation.

Bad: 
  * When one dimenion has a lot of data, performance is bad.
  * Efficiency degrades greatly with incresing dimension.
  
## Geo-hashing
  * [Geo-hash algorithm](http://www.cnblogs.com/LBSer/p/3310455.html)
    * encoding (lat,lng) into a String
    * the longer the encoded string, the better accuracy
    * can use prefix-matching to find POI
    * BinarySearch split to encode lag/lng seperately and then interleave them.
    * then encode with base32 string
  * Important properties
    * Long common prefixes indicating two places are near
    * Not all near-by places share common prefixes
      * need to expand to 8 near-by grids

poi | geo_hash_string (indexed) | lat | lng

procedure:
  1. Encoding geo-hash of queried (lat, lng) and prefix based on distance.
  1. Use query select * from pois where geo_hash_string like 'prefix%'
  1. Expand/Neighbour match using:
    select * from pois where geo_hash_string like prefix.expand() %
    * [How neighbour/expand is implemented](https://github.com/kungfoo/geohash-java/blob/master/src/main/java/ch/hsr/geohash/GeoHash.java#L384)

Good:
  * Caching friendly
  * ID index friendly
