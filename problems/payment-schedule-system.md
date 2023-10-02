# Payment scheduler system

## Problem statement

Design a system that allows users to schedule payments. This system uses an external API to perform the payment process.
```
    make_payment(payerId, payeeId, amount);
```


## Clarifying requirements and constraints

### Functional requirements

1. Users should be able to schedule one-time payments to other users / orgs
2. Users should be able to schedule recurring payments
3. Users should be able to view, edit and delete the payment schedules / events

### Non-functional requirements and assumptions

1. Users are everyday banking users
2. 2 M daily active users
3. Users just add schedules and forget. They access system more frequently during pay days end of the month and start of the month
4. Heaviest operation is to call the API from all the scheduled calls
5. During peak hours 10 M DAU - 3 scheduled events - 30 M total events per day - 347 write requests per second
6. Granularity of the event only extends to date - doesn't matter which time of the day amount needs to be paid.

## High level design

### API Design

* schedule_event(payerId, amount, payeeId, startDate, endDate, frequency);
* update_event(eventId, payerId, amount, payeeId, startDate, endDate, frequency);
* cancel_event(eventId);
* cancel_schedule(scheduleId);
* notification_service(payerId, payeeId);
* get_events(userId);
* make_payment(payerId, payeeId, amount);

### Database schema

* schedule(id, payerId, payeeId, startDate, endDate, isRecurring, amount, frequency)
* event(id, scheduleId, payerId, payeeId, date, amount)
* transaction(id, payerId, payeeId, amount, date)

### High level design

![payment-service-basic](https://i.imgur.com/KdsX6ka.jpeg)

## Detailed component design

### schedule_event
1. takes data from user - payee, amount, isRecurring, date and endDate if recurring
2. writes to DB in `schedule` table
3. also writes to db in `event` table for the specific date when the payment needs to be made.
4. in case of recurring payments - every freq cycle the amount is paid on the day when the schedule was first created.
5. for recurring payment schedules with no end date (`continue until canceled`), we create events for the next 12 or 18 cycles and create more later based on bandwidth.

### update_event
1. can update data for a specific payment event or the entire schedule
2. writes to and updates `schedule` and `event` tables

### get_events
1. gets the events for the user from DB and shows them in the UI
2. can store events of most active users in a cache for fast retrievals
3. LRU for cache eviction

### cancel_event and cancel_schedule
 
These services cancel the event and schedule respectively.

### payment_service
1. queries all the events need to be paid for the day
2. calls the external api for all the events - explore options for batch processing with the external api.
3. can batch events and execute them at regular intervals during the day since time of payment doesn't really matter.
4. for each successful payment record a transaction in db

## Bottlenecks and Tradeoffs

