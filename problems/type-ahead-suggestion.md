# Type Ahead Suggestion

Design a real-time suggestion service that recommends terms to users as they enter the text into input.

## Requirements and constraints

### Functional Requirements

Users should see top 10 word suggestions to the text they enter into the input box. 

### Non-Functional Requirements

1. Suggestions should appear in real time - updating with each text input change.
2. System should be reliable, scalable.

### Assumptions and Constraints

1. Read heavy system - all the queries are read operations with occasional write to the system in case there are a lot of searches for one particular phrase that doesn't exist in the system yet.
2. We can store the frequencies of each search phrase and when a term that is searched more than 1000 times and is not in the system, we can update the trie with that phrase.
3. Assuming 5B searches per day => 60k per second => 200B per search or 12MB per second throughput.
4. Assuming adding 10k new phrases per day => 2MB per day write => 2 * 365 * 5 = 3.56 GB

## High Level Design

### Components of the System

The system will have the following components -
1. Client - where users enter the text prefixes. sends the prefixes to backend every 100-200 ms to get a list of top suggestions to display in the client.
2. Data Collection App Server - collects data about top searches and provides data for suggestions to prefixes.
3. Get top suggestions App server - a server that processes the client's prefixes and sends the list of top suggestions
4. Storage - a form of DB that stores all the relevant data.


### Basic Algorithm / Data Structure

1. Since we need fast retrieval of strings starting with a part of the string, we can use a `Trie` data structure. A tree like data structure that stores a word with each letter in each node and nodes of one word as a subtree.
2. Ideally each node in trie will be a letter but to speed up the search and to save on storage space we can merge letters to more common combinations like `ca` instead of storing `c` and `a` in separate nodes. 
3. It is better to have a case-insensitive data structure as it would simplify the retrieval algorithm
4. Store trie in multiple servers to spread the load of all the search queries throughout the day.
5. Below is a node within the trie

```java
public class TrieNode {
    char val;
    Map<Character, TrieNode> children;
    boolean isWord;

    public TrieNode(char val){
        this.val = val;
        this.children = new HashMap<>();
    }
    public TrieNode() {}
}
```

### Top K suggestions for prefixes

1. We need to collect statistics of the search queries throughout the day, week, month and year. 
2. This would allow us to calculate the top suggestions for each prefix and also would allow us to update these suggestions based on new data.
3. We need to aggregate data on different levels of granularity. We can record the statistic data on the lowest granularity level and can aggregate them on different time periods. We can eliminate old data on lowest level to save space as soon as we get stats on other levels.
4. We can use Map Reduce algorithm and count-min sketch data structure to calculate the precise stats.
5. We can use a messaging system to streamline the processing of this data so the aggregation and creation of data are streamlined in order they are received.
6. We can use a SQL / NOSQL db to store this data for long term purposes.

Flow of this system -

1. User enters a term and presses enter on keyboard or clicks search on the screen - request goes to API gateway.
2. All requests to API gateway are logged. We can use these logs to build a hash table and store the data. These data can be aggregated at this point before being sent for more aggregation and storage.
3. Data sent to the messaging queue can be used to retrieve approximate but immediate top k results and / or can be used to create a more accurate top k results that would take more time. We can use the immediate top results to store data as a placeholder while use the accurate results when they are available.
4. Each path the data takes will be stored in the DB

### Storing and retrieving statistics of top k data 

1. We will aggregate the top search suggestions for each prefix and store them in our tries for fast retrieval. We can have dedicated servers that do this. 
2. We can store the top suggestions of each word within each character / sub part of the prefix.
3. This would increase the efficiency in finding top suggestions but would take more storage. 
4. We can improve the storage by storing the references of leaf nodes (complete phrases or words) instead of actual words. 
5. Each parent node will accumulate the top suggestions of their child nodes to get the top suggestions for that level. 
6. If we don't want to store the statistics of all time, we can use [Exponential Moving Average](https://en.wikipedia.org/wiki/Moving_average#Exponential_moving_average) system to give more weight to the recent stats rather than old ones. 
7. We can retrieve the top 10 terms for a prefix but just referring to the top suggestions part in the end node that represents the prefix.

### Storing the data

1. We need to store the trie efficiently and since everything depends on the data we need periodic backups of the trie.
2. We can store the data in a NOSQL DB as it is similar to a trie data structure and would ensure less retrieval time.
3. Each trie can be stored in a dedicated server


### High level design

![type-ahead-basic-design](https://i.imgur.com/00cPqcK.png)

### Request Flow

1. User types the characters, e.g. `ba`, on the client input box
2. Request is directed to app server which contacts storage server and retrieves the top suggestions for the prefix `ba`
3. These top suggestions are sent back to client in a response and they are displayed in a box below the input box.

## Data Partition and Replication

1. There can be a huge amount of search data and top suggestions data in the trie. Holding it all in one server / database will add a bottleneck.
2. So we can split the data based on different techniques - range based or capacity based
3. Range based would be based on the first character of the prefix, so something like `a-h` on one server, `h-m` on another etc. This has the drawback of having servers that are not evenly balanced because one range of terms can have loads more data than the other
4. We can alternatively partition based on the capacity of the server having a cap of storage on each one.
5. We need to replicate the data to avoid any single points of failure. So each trie server will have 2-3 replicas of data including suggestion data so even if the master goes down we can rely on replicas to have the data and have the system running as usual.
6. Data replication along with backups should give us enough reliability to fix things in case of any single points of failure

## Cache

We can use a cache server to speed up the retrieval of top suggestions for a prefix. We can store top 20% of days popular searches and that way can optimize the top suggestions retrieval response time.

## Detailed Design

![type-ahead-detailed](https://i.imgur.com/RLXo6pq.png)