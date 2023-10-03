# System Design Master Repo

A repository with all the preparation notes for system design. Course at - [Grokking the system design interview](https://www.designgurus.io/course/grokking-the-system-design-interview)

## Other courses / videos

1. [System Design Primer](https://github.com/donnemartin/system-design-primer)
2. [Problems](https://github.com/donnemartin/system-design-primer#system-design-interview-questions-with-solutions)
3. [Scalability lecture - Harvard](https://www.youtube.com/watch?v=-W9F__D3oY4)
4. [Scalability for dummies](https://web.archive.org/web/20220530193911/https://www.lecloud.net/post/7295452622/scalability-for-dummies-part-1-clones)

### Notes in the repo

1. [System Design Concepts](system-design-concepts.md)
2. [Problems](problems)
3. [Prep Questions](questions.md)

## System Design problems outline

1. __Outline use cases and constraints__ - clarify use cases and requirements with the interviewer. List functional and non-functional requirements. get back-of-the-envelope estimates for storage and network usage.
2. __System Interface Definition__ - define what some of the APIs that are expected from the system. Helps clarify the requirements further.
3. __Define Data Model__ - define a data model for various entities and how they interact with each other. this will help nail the flow of data in different scenarios. Also helps determine which kind of storage solutions to use.
4. __Create a high-level design__ - create a high level design that shows interactions between critical components
5. __Design core components__ - design what each core component should do
6. __Identify bottlenecks__ - identify potential problems - any single points of failure, data inconsistency etc.
7. __Scale the design__ - solve these bottlenecks by identifying the solutions (e.g. horizontally scaling servers, adding replica dbs etc.)

## Requirement clarification questions

It is better to clarify requirements than to jump in and solve the design.

![requirement-clarification-questions](https://i.imgur.com/4wBxnGh.png)

## System Design Interview

1. Essence of a System Design Interview is to produce a working design for a system - a design that makes sense and logically coherent.
2. The working design should scale and be able to assess reliability
3. There is no correct answer, this is kind of a collaboration between interviewer and interviewee. It is more about the flow of the interview than the exact answer

### Types of Interview Questions

1. Large distributed systems like facebook or youtube - these contain multiple interacting components like front-end, back-end, storage, network. This system needs to scale across many users.
2. single node design like design of an internal component - these questions focus on internal workings of a component, often scaling on one machine and focus on algorithms and data structures.
3. domain-specific design like design a network - these questions focus on a specific area like AI or networks etc. Questions often go deep into expertise.

### Common mistakes

1. Getting stuck in requirement gathering
2. Going too deep into a particular topic - don't spend too much time in one particular area
3. Skipping size / scale considerations
4. Skipping reliability
5. Taking a passive stance - need to lead the flow of the interview  

## Common Object-Oriented Design Patterns

1. Template - defines a structure for sub-classes in which steps of an algo and their order are defined. Example - inheritance in java, interfaces, sub-classes etc.
2. Strategy - this is similar to template pattern but the algorithm is chosen at run time rather than at compile time. Example - different compression algos selected based on the type of compression
3. Observer - in this pattern the observer objects subscribe to an observable object to be notified everytime there's any change in data. Two variants - __Push__ where the observable class pushes information to the observers and __Pull__ where the observers may pull the information from observable class. Push design puts control in observable class by making it the final controller of the information and the frequency of updates. Pull design puts control in observer classes by giving them the ability to customize aspects of information updates.
4. Singleton - ensures that only one instance of the object exists throughout the session. Example - Loggers
5. Abstract Factory - provides an interface for creating families of related objects without having to specify the concrete classes. Example - interfaces in java
6. Factory method - defines an interface for creating an object but lets subclasses decide which class to instantiate.
7. Builder - enables specification of fields that are needed for object creation. Makes code readable. Example - when an object needs to be created with different fields set in different circumstances
8. Prototype - uses a clone method to duplicate existing objects to be used as prototypes for new instances. Example - creating similar objects especially when creating new objects in expensive.
9. Adapter - allows interface of an existing class to be used from another instance. Example - when dealing with third party library code we often need to convert data from the type defined in our code to the type that works with the library code.
10. Flyweight - allows objects to be shared to reduce memory load. Example - object pools
11. Proxy - allows an object to act like a proxy - a lightweight version of the object and can be used in times of memory crunch. Example - like using a proxy image instead of real image for its metadata
