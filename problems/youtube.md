# YouTube

Design a video sharing service like Netflix or Youtube.

## Requirements

### Functional Requirements

1. Users should be able to upload videos
2. Users should be able to view videos
3. Users should be able to search for videos based on titles
4. Users should be able to see the number of views on videos

### Non-Functional Requirements and Assumptions
1. System should be highly reliable, available
2. Users should have real time experience

## Assumptions, constraints and Capacity Estimations
1. Assuming 500M daily users and most of them are video viewers and not creators - 500M * 5 videos per day => 2.5 billion videos per day or 28k videos per second
2. Read heavy system with view to upload ratio being 200: 1 - 28k/ 200 = 150 videos uploaded per second
3. Storing 150 videos per second with each video being around 50 MB means a total storage of 150 * 50 MB = 7.5 GB per second, which means a total storage of around 7.5 * 86400 * 365 * 5 = 1182600 TB for 5 years. Ignoring replication and compression
4. Bandwidth for upload = 150 * 50MB = 7.5 GB per second and for viewing = 20 * 75 = 1500 GB per second.

## High Level Design

On a high level the system would just have an app server serving all the requests to the client. We would use a DB to store data and object storage server to store videos and thumbnail images. We would need to add more components like encoder, message queue etc to make this design even more robust.

![youtube-high-level](https://i.imgur.com/F0OdDAB.png)

## Database schema

We need to store information about users, videos, video stats etc. Here is the schema for the same:

User - id, userName, email, name etc.
Video - id, title, description, userId, videoLink, thumbNailLink
VideoStats - id, videoId, likes, dislikes, views
videoView - id, videoId, timeStamp

Database choice - SQL / NoSQL

## Key Components

1. Encoder - We need an encoder to convert video into different formats, so we can choose the best format to play per device. Thumbnails for the videos can be uploaded by the users themselves or we can have the encoder generate them.
2. Message Queue - to streamline uploads, we can use a message queue like Kafka to queue uploads to encoder. A pipeline flow would be user uploads video => video and other info goes into message queue => encoder converts video into different formats and / or generates thumbnails => video and thumbnails are stored in object storage and video metadata stored in databases.
3. Object storage - videos and thumbnails are better stored in object storage systems like Amazon s3 or Hdfs.
4. CDN - to quickly deliver cached objects such as thumbnails and videos to client machines.
5. Cache - to store most-viewed / trending videos of the day and their metadata.
6. Load Balancers - to reduce load on a single server and to spread requests around to all app servers

## Detailed High Level Design

We would require around 28k requests / 1000 = 28 servers just to handle view traffic. We can have 100 servers to handle view traffic even during peak time. Adding the components above into the system we get this:

![youtube-detailed-design](https://i.imgur.com/ixlkEi8.png)
 
## Search and Views

### Search
1. A basic version of search would be to search through the database for video title and returning all the video titles that contain the keyword. 
2. We can index the column `title` in the video table to speed up the search operation. Search results contain, video links, thumbnails and other minimal metadata. 
3. In order to fasten search, we can denormalize relevant metadata within video table to avoid unions with other tables.
4. We can cache the search results for the top / trending searches of the day and retrieve these instead of hitting the database. 
5. We can maintain keyword information and update this information with video metadata and retrieve results that match with these keywords.

### Views, likes, dislikes

1. Views far outnumber other stats. Therefore, likes and dislikes can be stored in the video table without any additional calculation.
2. The views stat gets updated every second around 28k times. Updating the video table that many times is not viable. Therefore, we can record views in a separate table and periodically update the video table with total views.
3. We can record views of a video per second like a timestamp event and aggregate them to the video table. Since we are dealing with millions of videos and millions of views per video, this view table can fill up pretty quickly. 
4. Therefore, we can purge the view information of a video periodically or when the aggregation to video table is complete.

## Data Replication / partitioning

For obvious reasons all data cannot stay on one database server. We can have some data on a single server - like user metadata but even that is a bad idea. We can partition  data based on geographic location or based on ID. If done based on ID we need to find a reliable way to distribute data equally over all shards. 

If based on geographic location, we can further partition those geo-shards based on ID. Search function should then query all shards and aggregate results in a centralized server. 

Data should be replicated to avoid any single point of failures. A master slave architecture of shards is recommended. a lot of this functionality is inbuilt in databases like Cassandra.

## Reliability and Availability

Reliability and  Availability can be achieved with data replication like discussed above.