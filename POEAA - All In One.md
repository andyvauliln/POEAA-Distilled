# PATTERNS OF ENTERPRISE APPLICATION ARCHITECTURE

----------

# Introduction

## Thinking About Performance
* Response Time
* Responsiveness
* Latency
* Throughput
* Load
* Load Sensitivity
* Degradation
* Efficiency
* Capacity
* Scalability
	* Vertical Scalability - Scaling Up
	* Horizontal Scalability - Scaling Out

## The Structure of Patterns
* Name
* Intent
* Sketch
* Motivating Problem
* How It Works
* When To Use It (When Not To Use It)
* Further Reading

----------

# Chapter 1 - Layering
## Layering Benefits and Downsides
### Benefits
* Understand a layer independent of other
* Substitute layers - plug and play
* Minimize dependencies between layers
* Standardization

### Downsides
* Not everything gets abstracted, sometimes you have cascading changes
* Can harm performance

## The Evolution of Enterprise Applications
Mainframes => 2 Layers (Desktop Client, Server {DB}) => 3 Layers (Desktop Client, Domain Logic, {DB} Server) => 3 Layers (Web Interface, Domain Logic, {DB} Server)

## The Three Principal Layers
* Presentation : External interface for a service your system offers to an user or a client program
	* Web Client
	* Rich Client (Windows Based Application Clients)
	* Command Line or Text Based Menu System
* Domain Logic
	* A.K.A Business Logic
	* Calculation, Validation, Processing, etc.	
	* Sometimes the layers are arranged such that, domain logic completely hides data source from presentation.
* Data Source : Is the interface to things that provide a service to you
	* Database (SQL / NoSQL)
	* Transaction Monitors
	* Other Applications
	* Messaging Systems

> **NOTE** - Latest systems are developed such that presentation layer apps (desktop, laptop and mobile devices {tablets, phones and wearable}) talk to back-end services which in-turn use polyglot persistence.

### Separating Concerns
Level of separation purely depends on complexity and other requirements, but it is good to have some amount of separation at least at the subroutine level.

### Direction of Dependency
* Presentation -> Domain Objects -> Data Source
* Identifying and separating code across layers - If there is some functionality getting duplicated in a higher layer, that probably belongs to lower layers and it better abstracted as part of the lower layer. e.g. Adding a command line interface to a web application, if there is significant amount of code getting duplicated in order to do this, that's a sign where domain logic has leaked into presentation.

## Choosing Where to Run Your Layers
* Decision is whether to run on a client, or on desktops, or on servers.
* Running of server and having a simple HTML client has its own benefits.
	* Easy to upgrade or fix since it's in a limited amount of places.
	* No need to worry about deploying to desktops and keeping them in sync.
	* Don't have to worry about compatibility with other desktop software.
* Running on client provides some benefits as well	
	* Responsiveness
	* Disconnected / offline capabilities (now HTML5 has offline capabilities)
* Considering options layer by layer:
	* Data source typically runs on a server, that said, for providing disconnected capabilities, we need to have some amount of data store capabilities on the client and have a way to synchronize between offline store and data source server.
	* Presentation - This depends purely on the type of client you decide upon. Web clients vs Rich Client. As per latest trends (2016) we have plethora of devices interacting with back-end services. We also have Single Page Application with the recent avatar of JavaScript everywhere. Instead of relying in servers rendering the presentation, it is handled elegantly on the client-side by JS MV* Frameworks and Libraries. Thus the presentation, even if it is a web client, can run on both client and server. Rich clients as the name implies runs on client and leaves us open to challenges like version management, compatibility midst other issues. However we have another new ecosystem of clients -> "Apps" which again get classified as (Native, Web and Hybrid) running on mobile devices like mobiles, tablets, wearable and even automobiles. Hence take into consideration the requirements put-forth by business and their target audience / clients before deciding on the presentation.
	* Domain Logic - Best run on servers for reasons mentioned above as in data sources. However, a part of it could be on client for responsiveness or disconnected use.

> **NOTE** - Don't try to separate the layers into discrete processes unless you absolutely need to as doing that will both degrade performance and add complexity.

----------

# Chapter 2 - Organizing Domain Logic
* Three main patterns for organizing domain logic.
	* Transaction Script
	* Domain Model
	* Table Module

## Transaction Script
*  Script for an action or business transaction
* The disadvantages are more than the advantages like simplicity, ease of handling transaction boundaries and being similar to procedural model easily understood by developers.
* The code becomes an entangled mess as complexity increases.

## Domain Model
* Provides an Object Oriented way to handling domain logic.
* Build a model of the domain - validations and calculations are part of domain objects.
* Slightly hard to wrap your head if you are new, but once you make the paradigm shift and start using this, there is no going back!
* Handles increasingly complex logic in a well organized way.
* You still have to deal with database mappings - richness of your domain model is directly proportional to complexity of you database mapping (With the advent of NoSQL are there ways to handle this complexity?)
* A sophisticated data source layer is much like a fixed cost, it takes fair amount of time to build one or money to buy one.

## Table Module
* Important difference between this and Domain Model - In Domain Model, there is one instance per record in the database, whereas in case of Table Module there is only one instance per table - it is designed to work with a Record Set.
* Middle ground between Transaction Script and Domain Model
* Biggest Advantage - Fits into the rest of the architecture. GUI environments are build to work on the results of a SQL query organized in a Record Set.

## Making a Choice
Making a choice depends on lots of factors:
* If the complexity of the domain logic is more prefer using Domain Model. Identifying this complexity comes with experience working in multiple domains and also expertise in the given domain.
* Team's knowledge level and adapting capabilities to a great extent decides the choice. If the team makes the paradigm shift towards Domain Model with ease go for it!
* The choice is never cast on stone, only that it takes a while and it is tricky to refactor once the choice is made.

> **NOTE** - These patterns are not mutually exclusive choices.

## Service Layer
* Common approach to split the domain layer into two - Service Layer placed over an underlying Domain Model or Table Module. (Domain Layer using only Transaction Script doesn't warrant a separate Service Layer)
* Presentation Layer talk to this Service Layer and it acts like an API.
* Good spot for transaction control and security. (Example: Attributes in .NET used for describing transactional and security characteristics.)
* Key decision: How to much behavior to put in Service Layer?
* Typically oriented around use cases.
* Another extreme - most business logic is present inside Transaction Scripts inside the Service Layer. The underlying domain objects are very simple; if it's a Domain Model it will be one to one with the database and can thus use a simple data source layer such as Active Record.
* Midway - controller-entity style - have logic that is particular to a single transaction or use case placed in Transaction Scripts which are commonly referred to as controllers or services. Behavior that is used in more than one use case goes on the domain objects, which are called entities.
* If you are using Domain Model - Make It Dominant!
* Better to have thinnest Service Layer (OTOH Randy Stafford has seen success with richer Service Layer.)

----------

# Chapter 3 - Mapping to Relational Databases
## Architectural Patterns
* These drive the way in which domain logic talk to database.
* Pay attention: the choice made here has far reaching impact and is difficult to refactor.
* It's also a choice that is strongly affected by how you design your domain logic.
* Pitfalls using SQL
    * Developers don't understand SQL.
    * Problems in defining effecting SQL queries and commands.
    * DBAs also like to get at SQL to understand how to tune it and better arrange indexes for performance.
* Better to separate SQL from domain logic and place it in separate classes.
* Database developers have a clear place to go and find SQL and optimize it!
* Gateway - Base classes on the table structure of the database and have one class per database table.
* Types
    * Row Data Gateway
    * Table Data Gateway

### Row Data Gateway
* Have an instance for each row in the database table.
* Fits naturally into OO way of thinking about data.

### Table Data Gateway
* Have an instance for each table.
* Works on Record Set. Most UI applications and controls naturally work with a Record Set.
* As it fits nicely with a Record Set, it is the preferred choice while using a Table Module.
* Preferably use stored procedures instead of SQL, have a set of stored procedures as defining a Table Data Gateway. Also have in-memory Table Data Gateway to wrap calls to the stored procedures.

### Active Record
* If you Domain Model - both Row Data Gateway and Table Data Gateway could be used, that said, it might be too much of indirection.
* In simple applications - Domain Model, corresponds pretty closely to the database structure. (1 domain class per table)
* As these domain objects have moderately complex business logic, it makes sense to have each domain object responsible for loading and saving data - a.k.a Active Record.
* However, as Domain Model gets richer, 'Active Record' breaks down!
* Databases don't handle inheritance and things get more complex.
* As Domain Model gets richer Gateways can solve the problem, however, the coupling between Domain Model and Database remains and prevents us from testing our Domain Model in isolation without depending on the database.

### Data Mapper
* Isolate Domain Model from database using Data Mapper.
* Data Mapper handles loading and storing of data between databases and Domain Model and allows both to vary independently.
* Complicated database mapping architecture, yet has benefit of complete isolation of two layers.

> **NOTE** - Gateway is not recommended as the primary persistence mechanism for Domain Model. For a simple Domain Model - Active Record is the simplest way to go, for something complex use a Data Mapper.

### OODB - Object Oriented Databases
* There is a fundamental impedance mismatch between objects and databases. (Today NoSQL typically Document Databases helps handle this with ease upto some extent).
* There was some effort towards building Object Oriented Databases, you work with in memory objects the database determines when to move objects on and off disks.
* NoSQL and Polyglot Persistence are techniques used nowadays, which in a way is nothing but OODBs.
* NoSQL is gaining traction and more support. Earlier Object Databases had little support / adoption as compared to RDBMS.
* Implementations include - See [Wikipedia](https://en.wikipedia.org/wiki/Object_database#External_links)
    * Flat file
    * Column-oriented
    * Document-oriented
    * Object-relational
    * Deductive
    * Temporal
    * XML data stores
    * Triple-stores

### ORM - Object Relational Mapper
* Even if you cannot consider using OODB consider buying a ORM tool.
* Building a Data Mapper is a complicated endeavor
* These tools aren't cheap - Make the decision between buy Vs. build!
* Even if you buy these ORM tools, it is good to know these patterns, as it might help in tuning a tool as it takes a small but significant chunk of work.
 