Requirements:Let's say we work with the Election Commission. On Counting day, we want to collate the votes received at the lakhs of voting booths all over the country. Each booth has a voting machine, which, when connected to the network, returns an array of the form {[party_id, num_votes],[party_id_2, num_votes_2],...}. We want to collect these and get the current scores in real time. The report we need continuously is how many seats is each party leading in. Please design a system for this


# High level architecture
Four tiers split:
 1. Vote publisher
 2. Vote receiver & buffer/aggregation system
 3. Vote data persistance layer
 4. Vote Report layer to report time based report

* use Streaming/Messaging (Kafka, Kinesis) queue based async data publishing to connect tier 1 and tier 2.
* Each layer can use horizontal scaling with increasing number of nodes

## Vote publisher
a light weight publisher to publish vote message, the message could be
    {
        key : userId,
        timeStamp: currentTime
        votes: [party1, party2 ... ]
    }

## Vote receiver & buffer/aggregation system
Vote receiver can process a buffer of messages and emit List<Map<PartyId, Count>>

## Vote result persistance layer
Primary Key: PartyId
Sort Key: timestamp
Count: totalCount
(This part could be replaced with map-reduce job to process all the chuncks and load into this table as well. But In general there should not be significant number of shards)

## Log application layer
Primary Key: partyId,
Sort Key: timestamp
Count: totalCount
Query and append 'Vote result persistance layer' since last updated stamp.
