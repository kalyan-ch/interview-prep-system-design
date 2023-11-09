# Common patterns

This article outlines common patterns in desigining systems and what tools are best for some of those patterns.

* If we are dealing with a read-heavy system, it's good to consider using a Cache.
* If we need low latency in the system, it's good to consider using a Cache & CDN.
* If we are dealing with a write-heavy system, it's good to use a Message Queue for async processing
* If we need a system to be an ACID complaint, we should go for RDBMS or SQL Database
* If data is unstructured & doesn't require ACID properties, we should go for No-SQL Database
* If the system has complex data in the form of videos, images, files etc, we should go for Blob/Object storage
* If the system requires complex/heavy pre-computation like a news feed, we should use a Message Queue & Cache
* If the system requires searching data in high volume, we should consider using a search index, tries or a search engine like Elasticsearch
* If the system requires to Scale SQL Database, we should consider using Database Sharding
* If the system requires High Availability, Performance, & Throughput, we should consider using a Load Balancer
* If the system requires faster data delivery globally, reliability, high availability, & performance, we should consider using a CDN
* If the system has data with nodes, edges, and relationships like friend lists, & road connections, we should consider using a Graph Database
* If the system needs scaling of various components like servers, databases, etc, we should consider using Horizontal Scaling
* If the system requires high-performing database queries, we should use Database Indexes
* If the system requires bulk job processing, we should consider using Batch Processing & Message Queues
* If the system requires reducing server load and preventing DOS attacks, we should use a Rate Limiter
* If the system has microservices, we should consider using an API Gateway (Authentication, SSL Termination, Routing etc)
* If the system has a single point of failure, we should implement Redundancy in that component
* If the system needs to be fault-tolerant, & durable, we should implement Data Replication (creating multiple copies of data on different servers)
* If the system needs user-to-user communication (bi-directional) in a fast way, we should use Websockets
* If the system needs the ability to detect failures in a distributed system, we should implement a Heartbeat
* If the system needs to ensure data integrity, we should use Checksum Algorithm
* If the system needs to scale servers with add/removal of nodes efficiently, with no hotspots, we should implement Consistent Hashing
* If the system needs to transfer data between various servers in a decentralized way, we should go for
* Gossip Protocol
* If the system needs anything to deal with a location like maps, nearby resources, we should consider using Quadtree, Geohash etc
* Avoid using any specific technology names such as - Kafka, S3, or EC2. Try to use more generic names like message queues, object storage etc
* If High Availability is required in the system, it's better to mention that system cannot have strong consistency. Eventual Consistency is possible
* If asked how domain name query in the browser works and resolves IP addresses. Try to sketch or mention about DNS (Domain Name System)
* If asked how to limit the huge amount of data for a network request like youtube search, trending videos etc. One way is to implement Pagination which limits response data.
* If asked which policy you would use to evict a Cache. The preferred/asked Cache eviction policy is LRU (Least Recently Used) Cache. Prepare around its Data Structure and Implementation.
