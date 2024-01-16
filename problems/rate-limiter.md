# Rate Limiter

Design a rate limiter that would throttle user requests.

## Problem Statement

When a web application / API becomes highly popular or when one of the clients makes a lot more requests to the system than all the others combined, it makes sense to limit the number of requests so system can deal with reduced load and better serve the requests. This is called rate limiting or throttling. This allows to limit the number of requests a client can submit at a given time.

## Requirements, Assumptions and Constraints

### Functional Requirements

System should make a decision on each request - to allow or not

### Non-Functional Requirements

1. Low latency - make a decision on a request as quickly as possible
2. Accurate - need to accurately assess a request
3. Scalable - supports a large number of hosts in the clusters

## Basic Design

### Components

1. Rules Retriever - retrieves the rules from database - a rule specifies the number of requests allower for a particular client per second.
2. Rules Cache - stores the roles in local memory
3. Client Identifier - identifies the client from the request
4. Rate Limiter - checks if the request for a client has exceeded the limit based on rules in the cache. if not allows the request to go through. if exceeded, we reject the request and return a 503 or 429 error code or queue the request to be processed later.
5. Request processor - processes the request and returns a response.
6. Database - storage for the data

### Basic Design

![rate-limiter-design](https://i.imgur.com/B7qcnNT.png)

## Rate Limiter Algorithm

### Token Bucket algorithm

Initially a bucket (array or stack) can be full of tokens (random data). Each request will remove a token from the bucket and when the bucket is empty, any incoming requests are rejected. Buckets are filled at a constant rate. Basic implementation code below

```java

public class TokenBucket {

  private long maxBucketSize;
  private long refillRate;
  private double lastRefillTimeStamp;
  private long currentBucketSize;

  public TokenBucket(long maxBucketSize, long refillRate) {
    this.maxBucketSize = maxBucketSize;
    this.refillRate = refillRate;

    this.currentBucketSize = maxBucketSize;
    this.lastRefillTimeStamp = System.currentTimeInMillis();
  }

  // synchronized to remove race conditions between threads
  public synchronized boolean allowRequest(int tokens) {
    refill();
    if(currentBucketSize > tokens) {
      currentBucketSize -= tokens;
      return true;
    }

    return false;
  }

  public void refill() {
    long now = System.currentTimeMillis();
    double tokensToAdd = (now - lastRefillTimeStamp) * refillRate / 1e9;
    currentBucketSize = Math.min(currentBucketSize + tokensToAdd, maxBucketSize);
    lastRefillTimeStamp = now;
  }

}

```

## Scaling the Design

1. Scaling this system is adding more servers to rate limiter service and making all the requests go through a load balancer to evenly distribute this traffic.
2. Having multiple servers serve requests from clients will make this approach a bit complicated. Therefore we need to maintain token bucket on all servers. Throttling among distributed servers is achieved via inter server communication.
3. Servers can talk to each other about the number of tokens used so they know when to remove tokens from buckets in their respective machines. This will allow to not serve requests more than the limit agreed upon.
4. Talking to each other can be done using many different methods - everyone knows everything, gossip protocol, distributed cache etc.
