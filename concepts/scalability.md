# Scalability

* Scalability is the system's ability to handle increased load.
* Load can be described with a few numbers called load parameters. This depends on the system - could be number of requests per second, number of reads to writes in database, number of simultaneous active users etc.
* For example, for twitter distribution of followers per user might be a load parameter as this might determine the amount of writes when a user posts a tweet (twitter fan out). In this case the load parameter is number of followers since all of them need to get latest tweets to read.
* Performance is usually a service's response time or the number of records a batch process can process per second.
* Percentiles are a good metric to reliably know the typical response time of a service. 99th percentile metrics (meaning 99% of requests are faster than this metric) are usually high and companies use these values to set SLAs(Service Level Agreements) and SLOs(Service Level Objectives).
* Distributing load across multiple machines is known as shared-nothing architecture. But maintaining so many machines might become more complex and expensive. So an appropriate architecture for your system depends on the load metrics as defined above. There is no one-size-fits-all scalable architecture.
* For example, a system designed to handle 100k requests per second, each 1 kB in size, looks very different than one that is designed to handle 3 requests per second each 2GB is size. Architectures are built around assumptions of which operations will be commonly used and which will be rarely used.
