# Messenger 

Design an instant messaging app that users can utilize to send and receive text and picture messages.

## Requirements

### Functional Requirements

1. users should be able to send other users text messages.
2. users should be able to keep track of other users online / offline statuses
3. users should be able to send media messages like pictures / videos and audio files

### Non functional Requirements

1. Users should have a real time experience (minimum latency)
2. System should have high consistency - same data between different users and all the different devices
3. System should be reliable even for hundreds of millions of users

## Assumptions and Constraints

1. Around 500M users - 100 messages per day - 50 billion messages per day
2. 100 bytes per message - 5 TB of data sent and received per day - for each second the amount of data being sent and received is - 60 MB
3. if chat history of each person needs to be saved - 5TB * 365 * 5 = 9125 TB of storage space

## High Level Design

Users with their device of choice will connect to the chat server. When a user sends a message, the message will go to chat server and the server will send the message to the intended recipient. 

![messenger-design](https://i.imgur.com/YSfFsPz.png)

## Detailed Design

### Message flow

To get messages from the server, a user / client can utilize three options - 

1. HTTP Polling - client sends requests to the server frequently to check for any new messages. Server responds to one of the requests with the message information. So a lot of the requests that client sends are unnecessary. This results in sending unnecessary requests to the server and will cost in terms of bandwidth .
2. Long Polling - a modified version of the above where instead of sending so many requests, clients sends one and the server holds on to that requests and responds once any messages are available. This means that the client has an open connection with the server at all times. So once the server responds with data, the clients sends another request to open another connection with it. This is also faulty and has latency issues just like the approach above.
3. Web Sockets - in this approach client still maintains an open connection with the server, but it is a two-way connection. In this approach client and server both can send data to each other.

We can use Web sockets to send and receive messages, but we need a lot of servers to efficiently handle the load. Since we are designing a system for 500M users, we should have anywhere between 10K - 20K servers, assuming modern day servers can handle 50K requests at a time. If the users are offline and have some messages, we can store them until they're back and send the messages once the connection establishes

### Users' statuses

We can update the information about status of a user based on their connection status to the servers. We can send this information to each user whenever it changes using web sockets. However, constantly sending these updates to all clients might be a costly operation on the system. So one of the optimizations that can be done is to notify the status of userB only when userA opens up their profile / chat window. Alternatively we can update the database and clients can pull the information from the database about the last time user was active.

### Database Design

We need to store information about users at least if not the chats themselves. If chats are not needed to be saved to the database then we can store information about the users like ID, email, phone number etc. In this case, the chats would live in the client devices and need to be backed up (locally or to a cloud service) in case devices are lost, stolen or reset. 

If we need to store the chats, we need to link the chats to users. In this case, user chats can be restored even if client devices are compromised. This is how the schema will look like -

User - id, username, email, phone, lastActiveTime
Message - id, userFrom, userTo, text, mediaUrl, conversationId

Since there are no groups to support, we can ignore the downfalls of the message table design.

### Finalizing Design
 
Here is the final design with all the above points considered:

![messenger-final-design](https://i.imgur.com/1VGoTzu.png)

## Data Partitioning

## Cache
