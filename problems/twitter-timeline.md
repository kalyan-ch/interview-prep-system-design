
# Design twitter timeline / feed

## Step 1: Outline use cases and constraints

### Requirements

Overall these are the main functional requirements:

1. display top tweets and retweets from accounts that user follows
2. display tweets about topics that user is following / might be interested in
3. allow user to post a tweet - can contain text, images, videos, audio
4. search for a keyword or hashtag and show tweets for that keyword or hashtag
5. show a list of trending searches for the user

Non functional requirements:
1. Service should be highly available
2. Consistency can take a hit

### Constraints and Assumptions

* traffic is not evenly distributed - more reads that writes
* 100 million active users
* averaging 5 tweets per day per user - 500 million tweets per day
* fanning out a tweet to all followers means fanout of over 5 billion tweets a day
* 300 million searches per day
* 2.5 billion read requests per day
* Timeline should load fast
* optimize for faster reads vs writes
* search is read heavy too


* 500 * 10000 million bytes - 5TB * 365 * 5 - 10 PB per 5 years for tweets 
* 300k read requests per second
* 60k tweets to fanout every second
* 4k searches per second

## Step 2: System APIs and Data Model

These are some APIs that will be used in the system. Can use SOAP or REST architecture.

```
    tweet(tweet_data, media_ids)
    returns the url of the tweet or HTTP error in case of failure
    
    fanout(tweetId, followers)
    returns nothing but sends tweet to all the followers of the user who tweeted
```


### Database Schema

We need to store info about tweets, users, topics, user follows.
```
    tweet - id, userId, content, topicId, createdDate, mediaId
    user - id, email, dateOfBirth, favTopics, createdDate, lastLogin
    follow - followeeId, followerId
    media - id, tweetId
```

## Step 3: High level design

A high level design of twitter and various services

![twitter design](https://i.imgur.com/APhg4NZ.jpg)

## Step 4: Design core components

### Timeline service

* gets the latest tweets from people / organizations that user follows
* gets the latest tweets for the interests that user follows
* gets recommended tweets for the user
* a separate timeline object is created in cache and tweets for the user are stored in it
* this object resides in cache
* cache can get the latest tweets for a timeline object from write service and fanout service

### Write Service and Fanout Service

* allows users to post a tweet
* can store a tweet in relational DB according to the schema above
* since reads should be prioritized, write may not be that fast
* write should call fanout service so that all followers of the user get to see the tweet.
* write service should persist the tweet in cache first and db later
* there should be eventual consistency between cache and db
* can use object store to store media
* fan out service can send out notifications to followers for the new tweet (if subscribed)
* fan out service can see the users followers in the cache
* also can store this tweet in user timelines in cache

### Search Service

* users can search for a keyword / hashtag
* Show a list of trending searches - most searched words in the past 24 hours
* Store a mapping of keywords to Topics to consolidate many keyword searches into few.
* This list can be customized based on user location when each search can also have location information in it. 
* Search service can use a cache for results for frequently searched words / trending searches
* results need to be sorted in multiple ways - default being - most engaged tweets
* return a list of top 100 tweets for initial query and keep retrieving 100 more as the user scrolls

### Misc

* since one of our requirements is to have availability, we can add multiple servers to each service above and balance load using load balancers.
* since consistency is not a priority, we can first write data to a cache and eventually persist this data onto primary database and the replicas.
* since our traffic is read heavy, we can have dedicated db servers for read traffic only. all writes go to cache and then primary db and then will be replicated to secondary dbs and replicas.
* we can monitor each server and database and collect metrics to change the system accordingly. metrics like - at what time is the daily peak, how many tweets written / read per day, latency with each service etc. 

## Step 5: Identify bottlenecks and Scale the design

* Since we have at least 50k requests per second for each service we want to horizontally scale servers hosting the services
* In place of backend server in the above diagram we can add multiple load balancers to redirect requests to appropriate service and servers
* We can add multiple replicas of DB and a master-slave architecture to ensure data redundancy and to make the data highly available in case of failures
* We can partition the data into many buckets / shards. These buckets can be created based on userId, location or tweetId, or a combination of any of these.
* Fanout service might be a potential bottleneck. followers of popular users - e.g. celebrities, will notice significant delay in getting notifications for user tweets. 
* To remedy this, we can compute timelines with all tweets except from celebrities and show them to the users. At load time we can get the latest tweets from celebrties and merge it with the current timeline. 
* Evict inactive user (users who didn't login in the last week) timelines from the cache so that we have more space for more active users