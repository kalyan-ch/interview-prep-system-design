# Item Review System

Design a feature that accepts reviews from customers for an item ordered in a delivery app. Show the average rating for an item for each item under a store.

## Requirements

### Functional requirements

1. Users should be able to review items on their order. Review consists of rating value (number out of 5) and review text. 
2. Users should be able to see the average rating of reviews on each item when they visit a restaurant / store.
3. Users should be able to see top reviews for the item when they click on the item

### Non-functional requirements

1. System should be highly available so need not be consistent - eventual consistency should suffice 
2. System will be read heavy (10:1) as more users will read reviews / ratings as opposed to writing the same.

### Constraints and Scale determination

1. 100M DAU - 10 million daily writes / 100M daily reads
2. review - id(4), reviewText(255), rating(1), userId(4), itemId(4) - 270 bytes per review / 10M * 264 = 2.5 GB per day or ~10 TB for 10 years 
3. AverageRating - id, itemId, averageRating - 24 bytes * 30M items total = 1 GB   

## Data model

1. Review - id, userId, itemId, text, rating, totalUpvotes 
2. AverageRating - id, itemId, averageRating

## Core Components

These APIs will be used in the system.

1. write_review(userId, itemId, reviewText, rating);
   1. persists review to DB
   2. slow operation
2. rate_review(isHelpful);
   1. rates a review with either - is Helpful / not helpful
   2. helps gauge quality of reviews
   3. updates the review record in the record table with increment / decrement to the current value
3. get_avg_rating(itemIdList);
4. get_reviews_for_item(itemId);
5. calculate_average_rating(itemId);
   
## High Level Design

![item-review-system-hld](https://i.imgur.com/XFsJGqX.jpeg)

## Core Component design

The functionality of `write_service` and `rate_review` is straightforward - write data to DB directly. since the data need not be consistent, slow writes are fine. Since we will incorporate data redundancy, eventual consistency is achieved.

### get_reviews
* gets a list of reviews for an item
* is called when a user clicks on an item
* since this is one of high traffic operations, service doesn't have to retrieve data from DB and can get from cache
* cache contains reviews of food items of stores that are nearest to customers / popular stores in the area

### get_avg_rating
* retrieves the average rating of a list of food items
* this service can be called when user visits the store page or when user is viewing the popular dishes in the area
* since this is one of the high traffic operations, service uses cache for fast retrievals
* cache contains average rating of items that are popular in the area or that of items in the popular / nearby stores in the area.

### calculate_avg_rating
* periodically calculates average rating of every food item
* will process all data in chunks for efficient performance
* this can be a stored procedure living in the db that gets executed periodically
* updates table `AverageRating` with rating for food item

### Data Replication and Partition

Data needs to be replicated within the DB to avoid any single point of failure. DB can be designed into a master-slave architecture for data replication with asynchronous operations to update slave DBs with latest data.

Since review data is huge, it can be partitioned into multiple shards. They can be sharded based on location since we want to retrieve data based on user's location. For this purpose, we can persist location data along with review data. A typical shard contains data for a city, if it isn't too big. If it is, we can further partition based on zip code / county etc.  

## Scaling the design

We need to add load balancing in front of servers containing read services to evenly distribute traffic. Load balancer between client and `get_reviews` and another one between client and `get_avg_rating`. 

As already discussed, cache can be used to reduce latency in data read operation. Cache can contain reviews of all food items of all popular stores. Also, we can club the average rating information along with popular items information that already resides in the cache. LRU cache eviction can be used to get rid of stale data.