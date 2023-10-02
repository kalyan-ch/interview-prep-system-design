# Mint

## Problem statement
Design a website that tracks user expenses and creates budgets

## Outline use cases and constraints

### Functional Requirements
1. User links their financial account to Mint
2. A service extracts transactions from the account and :
   1. gets all the transactions in a month / quarter and adds it to DB
   2. categorizes all the transactions either into predefined or user defined categories
   3. analyzes transactions against the budgets set in place 
   4. daily updates to transactions
3. A service recommends budget to the users. 
   1. this budget can be adjusted by the user
   2. automatic budget can be recommended based on monthly or weekly income
   3. notifies users when approaching budget limit by category
4. Allows users to set goals - save money by a date, for example.
   1. can track goals against a bank account and a due date

### Non Functional Requirements

1. Website has high availability.
2. Transactions and user data must be secure

### Assumptions

1. traffic is not evenly distributed - more traffic towards updating transactions than adding accounts
2. daily updates of accounts happens only to most active users - only active in past 30 days
3. people rarely add / remove accounts - accounts may need to be synced when passwords are updated.
4. Sellers determine transaction category. type of seller can be categorized into
5. Write heavy functionality - updates to system / user daily but user visits and reads the analytics data less frequently. 

## Constraints and estimates

1. 10 million users - 4 accounts per user - 40 million accounts
2. 4 transactions per day - 40 million transactions per day - 500 per second
3. Each transaction - size 64 bytes, so 3 GB per day or 10 TB for 5 years
4. read requests - 50 per second

## Create High Level Design

A High Level Design

![mint design](https://i.imgur.com/FbuvR4O.jpg)

## Design Core components

#### Account Link Service

Links user account to Mint. Utilizes user credentials to establish connection. Stores the info in DB. If daily update to a specific account fails then sends notification to user to update the credentials to the account.

#### Transaction Read Service

Makes API call to the account using user provided credentials to retrieve all the transactions from all accounts for the day.

#### Categorize Service

Categorizes all the new transactions downloaded and calls notification service if any transaction has exceeded limit of budget set for the category. Also calls budgeting service for retrieving all budget categories.

#### Budgeting Service

Creates new budgets and categories - predefined based on popular trends and custom categories that user wants to define. User can adjust spending limit per category. Also retrieves list of budgeting categories for the user.

## Identify Bottlenecks

1. Transaction Read Service might be heavily used since it is called daily for 10 million subscribers.
2. Categorize Service is down the pipeline for transaction read service so even this suffers from heavy traffic.
3. Storing all data in a single DB might cause a problem in case of failure.
4. Retrieving data for budgeting categories etc. might cause heavy read traffic on DB.


## Scale the design

Some solutions we can implement to alleviate problems identified before:

1. Horizontally scale servers hosting TRS, CS, NS and BS - add more servers and reroute traffic using a load balancer.
2. Add replica DBs to add multiple copies of user transaction and account data.
3. Use a Master slave architecture to ensure high availability of data.
4. Use a Cache to store data of most frequently visiting users. Evict old data from cache.
