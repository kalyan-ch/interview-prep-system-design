# System Design Concepts

Here is a brief summary of most common system design concepts and topics.

## Key characteristics of distributed systems

1. __Scalability__ - Scalability is the capability of a system to grow and manage increased demand - increased data volume, increased amount of work etc., without losing any performance. Horizontal scaling is scaling by adding more machines to the existing pool. It is easier to scale dynamically this way than vertically. Vertical scaling is scaling by adding more power (RAM, Processor etc.) to the existing pool of machines. This type of scaling is limited by individual machine capacity.
2. __Reliability__ - Reliability is the probability a system might fail in a given period. A distributed system is reliable if it continues to deliver the services even if one of more software or hardware components fail. Achieving this might lead to redundancy. In an e-commerce store like amazon or walmart, no user transaction should cancel due to failing of any machines.
3. __Availability__ - Availability is the time a system remains operational to perform its function in a given period. Usually measured in percentages. Reliability vs Availability - if a system is reliable it is available. However, if it is available, it is not necessarily reliable.
4. __Efficiency__ - Efficiency is the performance of a given service or system.
5. __Manageability or Service-ability__ - This is the simplicity with which a system can be repaired or maintained. If this process is too complex or time taking, the availability of the system decreases. Ease of diagnosing, ease of making updates and ease of operation also factor into defining the Manageability of a system.


## Load Balancing

Load balancer spreads the traffic across the cluster of servers to improve availability and performance of a system. It avoids any server to be a single point of failure and keeps availability high by spreading the traffic. It should ideally keep track of all the servers and stop sending traffic to any servers that are down for whatever reason.

We can add a load balancer at each layer of the system to utilize full scalability: between clients and web servers, between web servers and app servers, between app servers and database etc.

Benefits of Load Balancing: Users experience faster and reliable service, service providers experience less downtime and higher efficiency, smart load balancers can predict bottlenecks and give actionable insights.

Load balancing algorithms chose a backend server by considering if the server is healthy. If there are many servers that are healthy one is chosen using:

1. Least Connection Method - server with the fewest active connections.
2. Least Responsive Time Method - server with the fewest active connections and least average response time.
3. Least Bandwidth Method - server with the least amount of traffic measured in Mbps
4. Round Robin Method - cycle through list of servers and issue request to the next server. starts from beginning if it reaches the end.
5. Weighted Round Robin Method - same as the above but each server is assigned a weight based on processing capacity and the LB selects next heaviest server
6. IP Hash - a hash of IP addresses of servers is maintained and request is redirected to one of them.

## Caching

Caching enables for vastly better use of resources we already have. Caches take advantage of locality of reference principle: recently requested resource is likely to be requested again. Caching can be used between every computing layer but often used near to the front-end layer.

Application Server Cache - a response data is stored in the cache and when the request is received the node checks the cache and returns it if found or it fetches the data from the service.

Content Distribution Network (CDN) - CDN serves large amounts of static media. CDN will serve the file if it is in cache or it will fetch the file from back-end service. Only used for large systems.

Cache invalidation - cache requires invalidation for removing wrong or old data. Following are the algorithms for effective cache invalidation:

1. Write-through cache - write data to cache and database at the same time. This has higher latency since write operation is performed more than once.
2. Write-around cache - write data to database directly bypassing cache. This method will require the cache to get data from database later and will cause "cache-miss"
3. Write-back cache - write data to cache only. perform database persistence from cache at regular intervals. This is efficient but there is a risk of data loss in the event of a crash or power failure.

Cache Eviction policies:

1. FIFO - cache evicts the first block of data accessed first without any other considerations.
2. LIFO - cache evicts the block accessed most recently without any other considerations.
3. LRU - cache evicts the least recently used items first
4. MRU - cache evicts the most recently used items first
5. LFU - cache evicts the least frequently used items first
6. RR - cache randomly evicts an item

## Data Partitioning

Data partitioning - split a database / table into many machines to improve Manageability, performance and availability. We need a load balancer here.

In Horizontal partitioning, the data is split based on a range of values, like all records with even ids in one table and odd ids in another. The split should be chosen carefully as the tables can be unevenly balanced. This is also called Data Sharding.

In Vertical partitioning, the data is split into different servers. Like storing user information in one database in one server, posts in another server etc. This is easier to implement and has low impact on the system. Depending on the size, the data within each server has to be split horizontally.

In Directory based partitioning, we have a directory server that abstract the partitioning schema away from the database and queries data based on a key-value kind of logic.

Partitioning Criteria:
1. Hash-based: a hash function is applied to the key attributes of data which yields a partition number. This approach is not flexible with the number of db servers changing.
2. List: each partition is assigned a list of values.
3. Round-robin: store data in partition `i mod n`, ith record and n partitions.
4. Composite: combination of any of the above

Problems:

a. joins on tables when data and databases are split, yield bad performance . A workaround is to denormalize the data but this would mean redundancy and inconsistency.
b. Enforcing referential integrity constraints could be hard. This would mean extra effort to clean or correct data.
c. There might be an effort to re-balance data on different servers, which means create more db servers or split huge tables across databases again.

## Database Indexes

Indexes are used in database tables to hasten up search queries. Indexes should only be used in tables if the tables are used more heavily for reading than for writing. Indexes increase the time taken to write to a table.

## Proxies

A proxy server is an intermediate piece of software or hardware that sits between a client and a server. A forward proxy hides the identity of the client from the server and a reverse proxy hides that of the server.

So, when you want to protect your clients on your internal network, you should put them behind a forward proxy; on the other hand, when you want to protect your servers, you should put them behind a reverse proxy.

## Redundancy and Replication

Redundancy is having more than one copy of critical components of the system. This typically increases reliability or performance of the system. This is one of the popular ways to eliminate single points of failure.

Database replication is process of copying and synchronizing data from one database to another. This increases availability, fault tolerance and scalability. Most of DBMS systems have a primary-replica relationship between different data servers. The updates to primary db get written to replica dbs as well. Popular replication strategies - synchronous - where primary db is replicated to the replicas before the write is considered complete, asynchronous - where db replication happens to the replicas at a later time. This kind of replication can have performance benefits. Semi-synchronous replication combines strategies from the above two.

## SQL vs NoSQL

In SQL databases, the data is stored in a structured fashion like, according to the principles of RDBMS. In NoSQL databases, data is stored in the form of documents, key-value pairs or graphs etc. which don't follow a definite structure.

SQL databases are schema-based (all data types and relationships should be determined beforehand), use some form of Structured Query Language (SQL) and are vertically scalable more easily than are horizontally scalable. NoSQL databases have dynamic schemas, use Unstructured query languages which differ with database and are easily horizontally scalable.

Since SQL databases are generally `ACID (Atomicity, Consistency, Isolation, Durability)` compliant, they are better in terms of safe transactions and data reliability. Since NoSQL databases are easily scalable and easier for development, they are better when an MVP needs to be pushed out fast.

## CAP Theorem

CAP Theorem states that it is impossible to achieve all three of  Consistency, Availability and Partition Tolerance in a single system.

1. Consistency - all nodes see the same data at the same time. users can write / read to any node in the system and see no inconsistencies in data.
2. Availability - all requests received by nodes must result in a response. Even when severe network failures occur, every request must terminate. Availability is the system's ability to remain accessible even if one or more nodes fail.
3. Partition Tolerance - a partition is a communication break between any two nodes in the system i.e. both nodes are up but can't communicate with each other. Partition-Tolerance means system continues to operate even if there are some partitions in the system. Such a system can sustain network failures as long as the entire network doesn't fail.

According to CAP theorem, any distributed system can only achieve two of the above properties - CA, AP, CP. CA is not a coherent option, as a system that is not partition-tolerant will be forced to fail on C or A properties. Therefore, a system must choose between Consistency and Availability.

## Consistent Hashing
Consistent Hashing

## Long-Polling vs WebSockets vs Server-sent Events

1. HTTP Polling - client sends requests to the server frequently to check for any new messages. Server responds to one of the requests with the message information. So a lot of the requests that client sends are unnecessary. This results in sending unnecessary requests to the server and will cost in terms of bandwidth .
2. Long Polling - a modified version of the above where instead of sending so many requests, clients sends one and the server holds on to that requests and responds once any messages are available. This means that the client has an open connection with the server at all times. So once the server responds with data, the clients sends another request to open another connection with it. This is also faulty and has latency issues just like the approach above.
3. Web Sockets - in this approach client still maintains an open connection with the server, but it is a two-way connection. In this approach client and server both can send data to each other.

## System Design Master Template

Biggest challenges in System Design interviews: knowing where to start and knowing if all the important parts were covered. Following topics can cover important concepts:

1. DNS - Domain name System is a fundamental component of internet infrastructure. It translates user-friendly domain names into IP addresses. Your computer sends a query to a recursive resolver which then searches a series of DNS servers followed by a Top-Level Domain server and ultimately the authoritative DNS server. Once the IP is located, the resolver returns it to the computer and allows it to browse to the website.
2. Load Balancer - see section above
3. API Gateway - this serves as a server that functions as an intermediary between external clients and internal microservices or API-based backend servers. API gateway routes incoming requests, manages authentication and authorization, limits and throttles the traffic depending on need, caches responses to avoid load on back end and reduce latency and transforms requests and responses.
4. CDN - a content delivery network is a distributed network of servers that store and delivery static and dynamic content to client machines that are geographically closer to it. See above section for more details
5. Proxy Servers - see section above
6. Caching - see section above
7. Data Partitioning - see section above
8. Database replication - see section above
9. Distributed Messaging Systems - they provide reliable, scalable and fault-tolerance means of exchanging messages between applications and servers
10. Microservices - an architectural style where the application is organized as an assembly of small loosely-coupled services that communicate with each other. Primary characteristics - single responsibility, independence, decentralization, communication and fault tolerance.
11. NoSQL dbs - see section above
12. DB Indexes - see section above
13. Distributed File Systems - storage systems designed to manage and grant access to files and directories across a network. Usually they are used to store files such as images, videos and other kinds of files.
14. Notification Service - notifies users with important events about the system or anything else
15. Full-text search - allows users to search for particular words or phrases within an app or a website. To accomplish this rapidly, an inverted index is used where words are associated with documents. Elastic search is an example.
16. Distributed Coordination services - these systems are engineered to regulate and synchronize the actions of distributed apps. They help in maintaining consistency. These systems are very helpful in large-scale distributed systems like in microservices architecture or clustered databases. Apache Zookeeper is an example.
17. Heartbeat - heartbeat services check the health of a machine and this helps in load balancing, repair and maintenance of the system. Usually server machines continually send heartbeat messages to a central monitoring service.
18. Checksum - checksum is used to validate the integrity of data. this is important with a lot of data moving between numerous components over a large network. Algorithms - MD4, SHA-1, SHA-256, SHA-512 etc.