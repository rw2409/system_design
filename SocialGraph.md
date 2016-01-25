https://code.facebook.com/posts/1691455094417024/graphql-a-data-query-language/
https://code.facebook.com/posts/227813470702374/tao-the-power-of-the-graph/
https://code.facebook.com/posts/225278320960049/the-underlying-technology-of-messages/

Feed:

Leaf:
* Map<User, List<Activities>>, activities are ordered by time
* Needs to run a service called "tailer" to respond to user activity append to logs
* Asynchriously back up to disk

Aggregator:
* From user find all its frirneds (potentially in leaf nodes as well)
* For each friend retrieve recent activities
* aggregate and rank feeds

Megafeed: publishing find broad cast pre-rank
    write amplication
    fan-out write


Multi-feed: pulling style
    flexible with real-time
feed architecture:
https://code.facebook.com/posts/781984911887151/serving-facebook-multifeed-efficiency-performance-gains-through-redesign/

API deisgn:
https://code.facebook.com/videos/236525183177572/how-to-design-great-apis-parse-developer-day-2013/


API design:
* intuition: similar as before, ugly thing should have indication
* documentation
* opionated
