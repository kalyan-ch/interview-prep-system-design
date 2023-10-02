# URL Shortening service

Design a URL shortening service

## Functional Requirements

1. Users should be able to enter URLs and get a shorter version of the URL
2. Shortened URL must be of the format - tiny.url/xByxRd - website name of our service followed by a random unique character combination.
3. When users click on the shortened URL it should redirect the web browser to the original URL that the user has entered.
4. Users should be able to pick custom character combinations

## Non-functional requirements

1. System should be highly available.
2. Low Latency to redirect to original URL

## Assumptions, Constraints and Capacity

1. Read heavy system - more users visit shortened urls every day than are writing them
2. Assuming 200M new URLs shortened per month -> 2 TB storage for 5 years, 2 billion reads per month => 46K reads per minute / 460 writes per second
3. Implement a cache that stores most frequently visited urls per day ~~ 20% of all requests per day ~~ 13 GB

## Key Components

1. shorten_url(url, userId, customAlias, expiry) => alias - generates an alias for the url that can be appended at the end of domain name to generate a shortened url of the format -`https://bit.ly/{alias}`. Alias needs to be as short as possible while also having enough space to generate a lot of unique combinations. We can generate a unique hash as an alias using any encoding algorithms such as MD5, Base64 etc. 6 digit aliases with base64 encoding ensures around 68 billion unique strings, while 8 digits make up more than 280 trillion unique strings. We can use 6 digit as that would sufficiently serve all the urls we can serve. This hash can also serve as key in the database to store and retrieve the urls. 
2. delete_url(alias)
3. lookup_url(alias, userId) => HTTP 302 redirect (location={url_from_db}) - when user enters the shortened url, system should search the DB and retrieve the original url based on the alias(hash) in the shortened url. 

## Database design

1. On a basic level the data that needs to be stored is just a key value pair
2. Therefore, we can use a NoSQL DB to store the pairs and user data

### Schema

user => id, loginDetails, trivial_cols
url => hash_value, url, expiry, userId

## High level design

![url-shortening-system](https://i.imgur.com/30Mxexz.png)

## Database partitioning

We can partition database based on many criteria. Since our data is mainly stored based on hash, we can partition them based on starting letters of the hash. We can also partition based on user's geographic location etc. 

It is important to notice different problems that come with data partitions, main problem being inconsistency. We can address inconsistency with strategies such as Consistent Hashing.  

## Data Replication

Since we prioritize availability over consistency we should make sure that data is replicated on multiple servers to eliminate single points of failures. We can utilize NoSQL dbs inbuilt functions to ensure data is replicated in the most efficient way possible. Replication can suffer when combined with partition so we need to make sure consistent hashing is implemented. 

## Cache

Implementing a Cache in the system can reduce the data retrieval times dramatically. Since our system is read heavy having a cache of all frequently visited data can reduce latency in serving the correct URLs. We can store top frequently visited hashes and store them in cache. We can use an LRU eviction policy to remove stale data from cache. 