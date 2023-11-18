# System Design Concepts

Here is a brief summary of most common system design concepts and topics.

## Key characteristics of distributed systems

1. __Scalability__ - Scalability is the capability of a system to grow and manage increased demand - increased data volume, increased amount of work etc., without losing any performance. Horizontal scaling is scaling by adding more machines to the existing pool. It is easier to scale dynamically this way than vertically. Vertical scaling is scaling by adding more power (RAM, Processor etc.) to the existing pool of machines. This type of scaling is limited by individual machine capacity.
2. __Reliability__ - Reliability is the probability a system might fail in a given period. A distributed system is reliable if it continues to deliver the services even if one of more software or hardware components fail. Achieving this might lead to redundancy. In an e-commerce store like amazon or walmart, no user transaction should cancel due to failing of any machines.
3. __Availability__ - Availability is the time a system remains operational to perform its function in a given period. Usually measured in percentages. Reliability vs Availability - if a system is reliable it is available. However, if it is available, it is not necessarily reliable.
4. __Efficiency__ - Efficiency is the performance of a given service or system.
5. __Manageability or Service-ability__ - This is the simplicity with which a system can be repaired or maintained. If this process is too complex or time taking, the availability of the system decreases. Ease of diagnosing, ease of making updates and ease of operation also factor into defining the Manageability of a system.

## Reliability

For a software application, the expectations are:

* It performs the function that the users expect.
* It can tolerate users' mistakes or unexpected uses of it
* Its performance is good enough under the expected load and data volume.
* It prevents any unauthorized access and other security risks.

For an app to be reliable, all of these conditions should be met even when "things go wrong" or faults.

* Such apps or systems are called fault-tolerant.
* A fault is different from failure as any component can be faulty if it deviates from its spec.
* It will help to introduce faults deliberately as this will ensure the efficient design of a fault-tolerant system. For example, [Netflix Chaos Monkey](https://netflixtechblog.com/the-netflix-simian-army-16e57fbab116?gi=8ab06b5f6b85).
* Sometimes, prevention of faults is better than tolerating them like in cases of security faults.
* Hardware faults, like RAM faults, Hard Disk crashes, power outages etc., can be handled with redundancy. Hard Disks may be setup in a RAID system, systems may have mutliple power supplies or backup generators.
* However, redundancy isn't alone enough to prevent all faults. Therefore, systems are being designed to tolerate loss of entire machines by using software tolerant mechanisms, like using entire alternate data centers or rolling upgrades.
* Software faults, like bugs are much harder to anticipate and takes longer to fix, like [leap second bug in Linux Kernel](https://www.somebits.com/weblog/tech/bad/leap-second-2012.html) or deadlock bugs with locks on shared resources.
* Thorough testing, process isolation, measuring and monitoring system behavior in production etc. can help fix these bugs.
* Human errors like bad design of systems, configuration errors etc. also cause faults. They can be fixed by designing efficient systems, isolating failure parts of the system, thorough testing in all levels of the system, thorough monitoring, good management practices and training etc.
* Reliability is very important to preserve user data as poorly managed faults may result in huge losses of revenue to a company as well.

## Scalability

* Scalability is the system's ability to handle increased load.
* Load can be described with a few numbers called load parameters. This depends on the system - could be number of requests per second, number of reads to writes in database, number of simultaneous active users etc.
* For example, for twitter distribution of followers per user might be a load parameter as this might determine the amount of writes when a user posts a tweet (twitter fan out)
* Performance is usually a service's response time or the number of records a batch process can process per second. Percentiles are a good metric to reliably know the typical response time of a service.
* 99th percentile metrics (meaning 1 in 10000 requests) are usually high and companies use these values to set SLAs and SLOs.
* Distributing load across multiple machines is known as shared-nothing architecture. But maintaining so many machines might become more complex and expensive. So an appropriate architecture for your system depends on the load metrics as defined above. There is no one-size-fits-all scalable architecture.
* For example, a system designed to handle 100k requests per second, each 1 kB in size, looks very different than one that is designed to handle 3 requests per second each 2GB is size. Architectures are built around assumptions of which operations will be commonly used and which will be rarely used.

## Maintainability

* Maintenance of software contributes to the majority of its cost - fixing bugs, adapting to new platforms, modifying for new use cases, adding new features etc.
* We should design software in such a way that it will make maintenance easier. These design principles should help:
  * operability
  * simplicity
  * evolvability
* Operability indicates the ease with with the operations team can keep running the system smoothly. Automation should be created to ensure that the software system works correctly.
* A good operations team is reponsible for - monitoring the health and quick restoration of the system, tracking down root causes for system failures or degraded performance, keeping software platforms up to date including security patches, keep track of how different systems affect each other, capacity planning, establishing good practices for deployment, config management, performing complex maintenance tasks, maintaining security of the system with different changes and preserving the knowledge of the system and operations.
* Software that is operable with ease will allow operations team to focus the efforts on high-value activities. Routine tasks can be made easy by doing things such as providing visibility into runtime behavior, providing automation support and integration, avoiding dependency on individual machines for decent performance, providing good documentation, self healing when appropriate etc.
* Simplicity indicates the ability for new engineers to understand the system by removing complexity around the system. Complexity in a software project can have various symptoms - tight coupling of modules, tangled dependencies, inconsistent naming, hacks instead of design changes, adding many special cases for many issues etc. Complexity makes everything hard - budgets, scheduling, more likelyhood of bugs, maintenance cost etc.
* Making a software system simpler doesn't mean to reduce functionality - it can mean to remove accidental functionality. [Moseley and Marks](https://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.93.8928) define complexity as accidental if it is not inherent to the problem that the software system solves. One tool to remove complexity is abstraction - it can hide a great deal of implementation behind simple classes. It also allows software classes and methods to be reused.
* Finally, evolvability indicates the ease with which software system can change to fit future requirements or changes in existing requirements. Organizations use Agile methodology and patterns such as TDD, refactoring to adapt to frequent change. 

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

* joins on tables when data and databases are split, yield bad performance . A workaround is to denormalize the data but this would mean redundancy and inconsistency.
* Enforcing referential integrity constraints could be hard. This would mean extra effort to clean or correct data.
* There might be an effort to re-balance data on different servers, which means create more db servers or split huge tables across databases again.

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

1. In data partitioning, it is challenging to maintain and preserve data whenever a shard of data is added or removed.
2. A simple hashing is hard to maintain because if new servers are added or when existing ones are removed, we need to rebuild the entire hash function and server keys to evenly distribute data.
3. Consistent hashing uses keys and server names in hash function to map data, so it is easier to maintain.
4. A hash ring is formed with all the possible hash numbers (range) and servers and keys are hashed and placed on this ring. To map keys to servers, we go clockwise on the ring - key `k1` is mapped to the next server `s1` on the ring clockwise. So when new server is added or removed, only a few keys are remapped, thereby reducing the overhead to maintain data.
5. This isn't perfect either - data can be imbalanced. Virtual nodes can be used to represent a server to balance the data more efficiently.

Reference video - [Consistent Hashing](https://www.youtube.com/watch?v=UF9Iqmg94tk)

## Long-Polling vs WebSockets vs Server-sent Events

1. HTTP Polling - client sends requests to the server frequently to check for any new messages. Server responds to one of the requests with the message information. So a lot of the requests that client sends are unnecessary. This results in sending unnecessary requests to the server and will cost in terms of bandwidth .
2. Long Polling - a modified version of the above where instead of sending so many requests, clients sends one and the server holds on to that requests and responds once any messages are available. This means that the client has an open connection with the server at all times. So once the server responds with data, the clients sends another request to open another connection with it. This is also faulty and has latency issues just like the approach above.
3. Web Sockets - in this approach client still maintains an open connection with the server, but it is a two-way connection. In this approach client and server both can send data to each other.

## Bloom filters

1. Bloom filters are a hash table like data structure that help us find if a key is in them, but quickly.
2. This operation is not 100% accurate as it could result in false positives i.e. the operation can say that a key exists even if it doesn't. However, it would never result in a false negative. If a key doesn't exist, it always returns that information with 100% accuracy.
3. Bloom filters are used when accuracy isn't of high priority but speed is. For example, in NoSQL databases instead of searching millions of documents for existence of keys, using a bloom filters data structure ensure that this operation is efficiently fast.
4. The underlying data structure consists of buckets of bits, all unset (an array works too for small use cases). Keys are put into a hash function that returns a list of bucket numbers (indices) that are set. Any function that is checking for existence of keys need to just look at set indices to verify the same.

## Message Queues

message queues

## CDN

1. CDN - content delivery network is a network of servers that are geographically distributed so that content can be served to the users from data centers that are closest to them.
2. Data centers are called - Point of Presence (PoP) and servers in Pops are called edge servers.
3. CDNs serve as caches that reduce workload on core servers of the company / service. They usually cache static content. They usually query the origin servers for latest updates and store the cache that is relevant to the user of that geographic location.
4. For example, Netflix can use CDNs to serve the files for top videos of the day and this data will change with region like CDN in India will have more indian language videos whereas CDN in US will have more hollywood ones.
5. CDN also improve performance and availability of the service since they are fundamentally distributed.

## System Design Master Template

Biggest challenges in System Design interviews: knowing where to start and knowing if all the important parts were covered. Following topics can cover important concepts:

1. DNS - Domain name System is a fundamental component of internet infrastructure. It translates user-friendly domain names into IP addresses. Your computer sends a query to a recursive resolver which then searches a series of DNS servers followed by a Top-Level Domain server and ultimately the authoritative DNS server. Once the IP is located, the resolver returns it to the computer and allows it to browse to the website.
2. API Gateway - this serves as a server that functions as an intermediary between external clients and internal microservices or API-based backend servers. API gateway routes incoming requests, manages authentication and authorization, limits and throttles the traffic depending on need, caches responses to avoid load on back end and reduce latency and transforms requests and responses.
3. Distributed Messaging Systems - they provide reliable, scalable and fault-tolerance means of exchanging messages between applications and servers
4. Microservices - an architectural style where the application is organized as an assembly of small loosely-coupled services that communicate with each other. Primary characteristics - single responsibility, independence, decentralization, communication and fault tolerance.
5. Distributed File Systems - storage systems designed to manage and grant access to files and directories across a network. Usually they are used to store files such as images, videos and other kinds of files.
6. Notification Service - notifies users with important events about the system or anything else
7. Full-text search - allows users to search for particular words or phrases within an app or a website. To accomplish this rapidly, an inverted index is used where words are associated with documents. Elastic search is an example.
8. Distributed Coordination services - these systems are engineered to regulate and synchronize the actions of distributed apps. They help in maintaining consistency. These systems are very helpful in large-scale distributed systems like in microservices architecture or clustered databases. Apache Zookeeper is an example.
9. Heartbeat - heartbeat services check the health of a machine and this helps in load balancing, repair and maintenance of the system. Usually server machines continually send heartbeat messages to a central monitoring service.
10. Checksum - checksum is used to validate the integrity of data. this is important with a lot of data moving between numerous components over a large network. Algorithms - MD4, SHA-1, SHA-256, SHA-512 etc.
