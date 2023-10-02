# Instagram

## Problem statement
Design a photo-sharing site like Instagram.

## Requirements

### Functional Requirements

1. Users can upload photos / videos
2. Users can follow other users
3. Users can view a news feed that generates all the top photos of the users that they follow

### Non-functional requirements

1. Website needs to be highly available
2. News Feed generation needs to be fast < 200 ms
3. Website should be reliable

### Assumptions

1. Website is read heavy - users view photos / videos more than they upload
2. Consistency can take a hit since availability is prioritized. User need not see latest photos but website needs to be available
3. Need two kinds of storage - object storage to store media files and db for storing data related to users and metadata on media

## Capacity Estimation

1. 10 M DAU - 2 M photo/video uploads per day
2. Space required to store a day of photos or videos - 2M * 1 MB = 2 TB / 2 * 3650 = 7300 TB for 10 years
3. 10:1 read/ write ratio = 20 M read requests per day or 231 per second
4. 500M - 1B total users - 10 kb of metadata per user - 64 GB of metadata
5. Photo - user link table - 5 * 365 - 2 TB
6. Follower table - 4 TB

## High Level Design


## Data model / schema

User - id, name, email, phone, dateOfBirth, createdDate, lastLogin
Photo - id, path, lat, long, createdDate
UserFollow - followerId, followeeId

## Component Design

create new profile - create_profile(name, email, phone, dateOfBirth) => boolean;
* takes in required data for a new profile and returns a boolean value indicating if the operation is a success
* stores user data in user table
* can directly write to db as this process doesn't happen too often and we can afford a slow write

media upload service - upload(userId, media_bytes, lat, long) => void;
* upload writes to db and storage - a slow process
* can have dedicated servers running this service
* can write to cache first and eventually write to db and storage if too many upload requests are being handled at the moment.

timeline service - () => List of media;
* fetches the latest and most popular photos / videos of the users that the current user follows.
* can fetch the first 100 pics / videos and can fetch the next 100 as the user scrolls.
* gets a list of users that user follows and gets top 5 media for each user.
* a ranking algorithm can then decide which 100 of these can be returned to the caller.
* This service can be called in background and have the results stored in a cache, so when a user opens the app, the news feed is pre-generated for them.
* Service also needs to work with sharded data. 

## Reliability and data replication

* Since reliability is one of the core requirements, we need to make sure we don't lose any files or data.
* For this purpose, we can replicate data in both db and file storage servers. 
* Can have a master slave architecture so there are always copies of data. 
* This removes any single point of failure.

## Data partition

* Since we have huge amount of data, we need to effectively store them so that one server doesn't get all the traffic. 
* We can partition the photo-user data based on location and retrieve the data based on user's location. 
* We can partition further based on photoIds, userIds etc. Add a proxy server on top of db layers so that requests can be directed towards the correct shard.
* We can apply these techniques to file storage servers too.

## Scale the design

* Utilize a load balancer to manage load between different servers
* Utilize a cache for fast retrievals of data - use LRU to evict cache
* Utilize a CDN for media so that users get the media from geographically close CDN servers
