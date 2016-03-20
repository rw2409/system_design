[Probalistic Data Structures](https://highlyscalable.wordpress.com/2012/05/01/probabilistic-structures-web-analytics-data-mining/)
1. Linear counting: single function bloom filter: Cardinality Estimation
1. LoglogCount: Cardinality Estimation
1. Count-Min Sketch: Frequency estimate
  * Estimations for relatively rare values can be imprecise, but frequent values and their absolute frequencies should be determined accurately.
1. Count-Mean-Min Sketch: Frequency Estimate
  * reduce the average mean to achieve better performance
  Heavy Hitters: count-Min-Sketch + HEAP

1. Bloom Filter: Membership Query
  * only contains false positive
  * https://en.wikipedia.org/wiki/Bloom_filter
