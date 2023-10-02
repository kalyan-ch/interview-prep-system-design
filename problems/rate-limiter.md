# Rate Limiter

Design a rate limiter that would throttle user requests.

## Requirements, Assumptions and Constraints

### Functional Requirements

1. System should make a decision on each request
2. this decision is to allow a request to be processed or not.

### Non-Functional Requirements

1. Low latency - make a decision on a request as quickly as possible
2. Accurate - need to accurately assess a request
3. Scalable - supports a large number of hosts in the clusters

## Basic Design and components

