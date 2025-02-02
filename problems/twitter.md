
# Design twitter timeline / feed

## Outline use cases and constraints

### Requirements

Overall these are the main functional requirements:

1. Create an account
1. Send a text, image or video based post called tweet.
1. Like tweets
1. Follow other users
1. View timeline - a list of top tweets from the people that the user follows

Non functional requirements:

1. Service should be highly available
2. Need not be immediately consistent, i.e. not all users need to see the latest tweets right away (Eventual Consistency)

### Constraints and Assumptions

* traffic is not evenly distributed - Read Heavy
* 100 million active users
* averaging 5 tweets per day per user - 500 million tweets per day
* fanning out a tweet to all followers means fanout of over 5 billion tweets a day
* 300 million searches per day
* 2.5 billion read requests per day
* Timeline should load fast
* optimize for faster reads vs writes
* search is read heavy too
* 500*10000 million bytes - 5TB*365 * 5 - 10 PB per 5 years for tweets
* 300k read requests per second
* 60k tweets to fanout every second
* 4k searches per second

## System APIs and Data Model

These are some APIs that will be used in the system. Can use SOAP or REST architecture.

* tweet(tweet_data, media_ids) - returns the url of the tweet or HTTP error in case of failure
* fanout(tweetId, followers) - returns nothing but sends tweet to all the followers of the user who tweeted
* likeTweet(tweetId, userId) - returns boolean to indicate if the like was successfully saved in db

### Database Schema

We need to store info about tweets, likes on the tweets, users, user follows.

![twitter-basic-schema](https://i.imgur.com/je5UlUZ.jpeg)

## High level design

A high level design of twitter and various services
![twitter-basic-design](https://i.imgur.com/NLKAtMX.png)

## Core components

### Timeline service

* Gets the top tweets from people that user follows
* Top tweets might mean latest / recently popular tweets
* This is just a SQL join query between the tables - tweets, user and likes. This will work for few users but as number of users grow this query becomes slower and slower.
* For this we can pre generate news feed and store them in a separate DB / cache.
* A separate timeline object is created in cache and tweets for the user are stored in it. This one can be a NoSQL document style object that contains the following fields - userId, topPosts which is a list of documents containing - textContent, mediaURL, likesCount, userName, userId, createdDate among other fields.
* This NoSQL can be used to get the timeline tweets for a user, instead of running a SQL join query to get the tweets, userInfo etc.
* Instead of getting the timeline info everytime user opens the home page, we can store a few different timeline objects and send them to the client. This will reduce the number of calls made to cache / app servers.
* Cache can also get the latest tweets for a timeline object from write service and fanout service

### Write Service and Fanout Service

* Allows users to post a tweet with text and optional video / photo.
* Since reads should be prioritized, we don't need to block the UI until the tweet is saved.
* Since media is involved, the video / photo files need to be encoded and stored in the file system. The save to database process should wait for the mediaURL value to generate, so it can save this value to database for the relevant tweet.
* After the save is successful, the writeService should call fanoutService so that all followers of the user get to see the tweet.
* fan out service can send out notifications to followers for the new tweet (if subscribed)
* fan out service can see the users followers in the cache
* also can store this tweet in user timelines in cache

## Identify bottlenecks and Scale the design

* We have 500 million tweets per day. All of that data in one table and one database is a huge bottleneck. We need data partitioning to effectively store and retrieve the data.
* Calling backend app servers and database for each bit of information will overload the system. So having a cache that can store most frequently accessed tweets and their media will reduce some of the network load.
* Since we have at least 50k requests per second for each service we want to horizontally scale servers hosting the services, so add load balancer and many app servers to equally distribute network load.
* Having a single database and file system store will cause availability issues in case either or both of them go down. So we need to replicate the data into multiple databases and file systems.
* Fanout service might be a potential bottleneck. followers of popular users - e.g. celebrities, will notice significant delay in getting notifications for user tweets.
* To remedy this, we can compute timelines with all tweets except from celebrities and show them to the users. At load time we can get the latest tweets from celebrties and merge it with the current timeline.
* Evict inactive user (users who didn't login in the last week) timelines from the cache so that we have more space for more active users.

### Data partioning and Replication

* Paritioning data needs to be done for efficient performance of the system.
* We can partition in many different ways
  * based on time tweeted, which means old tweets are moved into a set of partitions that are less frequently accessed
  * based on geographical location, which means storing tweets from one region in a local partition.
  * based on tweet content, storing tweets with text in a set of partitions and those with media in others.
* A hybrid approach probably works best, since we can combine many techniques to effectively store the tweets. For example, further partitioning on tweet creation time, after creating partitions on geographical location.

### Cache

We can introduce a cache layer to store frequently accessed data. This means, we can store hot tweets and their information in the cache, which would reduce network load on app servers and database.

We can employ LRU to evict the least recent used data and store more relevant tweets in cache. This would mean we would always have trending tweets in our cache.

## Extended Requirements and their high level design

* Retweet - a feature where a user can endorse a tweet as if posting from his account.
* Hashtags - a searchable link that can indicate tweets about one or more topics.
* Trending topics - a list of topics that everyone on the app is talking about. This would improve engagement for the topic.
* Search - search tweets / users based on text.
