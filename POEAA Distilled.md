# PATTERNS OF ENTERPRISE APPLICATION ARCHITECTURE DISTILLED

> **IMPORTANT NOTE** - This work contains notes gathered by reading Martin Fowler's book. This includes important points from the book and also compares the concepts / patterns with latest developments in the area of enterprise application development, based on the limited experience and knowledge, that I possess. - **Srihari Sridharan**

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

## The Behavioral Problem

* The behavioral problem is how to get the various objects to load and save themselves into the database.
* Simple route - an object to have load and save methods to database - with Active Record, this is an obvious route to take.
* Typical problems:
	* Keeping track of data loaded from database, data added / edited / deleted.
	* As objects are read, ensuring that database state you are working with stays consistent.

### Unit of Work 

* Unit of Work is a pattern that's essential to solving both these issues.
* Tracks all the creates, updates and deletes.
* Instead of application programmer invoking explicit save methods, tells the Unit of Work to commit.
* Unit of Work sequences all the appropriate behavior to the database.
* This pattern is essential when behavioral interactions with database becomes awkward.

### Identity Map

* As objects are loaded - ensure same object is not loaded twice.
* Issue - 2 in memory objects corresponding to the same database row.
* To deal with this issue - maintain keep record of every row you read in an Identity Map.
* Each time you read, check the Identity Map, if found, return a reference, if not go ahead and read from database and create and object.
* Identity Map also doubles as a cache for database.

> **NOTE** - The primary purpose of an identity map is to maintain correct identity and not boost performance by acting as a cache.

### Lazy Load

* In case of Domain Model, you will usually arrange things so that, linked objects are loaded together in such a way that a read for an order object, loads its associated customer object.
* However you might pull an enormous amount of data in case of a huge object graph, which might hurt performance.
* Use Lazy Load and reduce what you bring back, but still keep the door open to pull back more data.
* Uses a place holder reference and later pulls data from database based on need and points to actual object reference.

## Reading Data

* Finder methods - wrap SQL select statements with a method-structured interface.
* E.g. find(id), findForCustomer(customer)
* Where you put these depends on interfacing pattern used.
	* In case of Table Data Gateway (one class per table) these find methods are can be included alongside update and delete methods in the same class.
	* In case of Row Data Gateway - you need a separate class to abstract these find methods.
	* Finder objects typically return an appropriate collection of row based objects.
* Ensure that find methods work on database state and not on object state. Do queries at the beginning.
* Performance considerations as listed below:
	* Pull back multiple rows at once.
	* Do not issue repeated queries on same table to get multiple rows.
	* Use joins to pull multiple tables back with a single query.
	* Mind that - databases are optimized to handle up to 3 or 4 joins per query.
	* Used cached views to improve performance.
* Database optimization:
	* Clustering common reference data together
	* Careful use of indexes
	* Database's ability to cache in memory.
	* Reach out to DBA!
* Very important do performance profiling and tuning and then take a call, as it depends purely on environment that is used in a project!

> **NOTE** - It is tempting to use *Static* methods to implement find operations. However doing so prevents database operations being substitutable and that means you cannot use a Service Stub. Hence it is advisable to have a separate finder objects.

## Structural Mapping Patterns

* Patterns used while mapping between in-memory objects and database tables.
* Aren't relevant to Table Data Gateway
* Few of them are used in case of Row Data Gateway and Active Record
* Probably all of them are used for Data Mapper.

### Mapping Relationships

* Central issue - way objects and relations handle links.
	* First, difference in representation.
		* Objects handle links by storing references that are held by the runtime of either memory managed environments or memory addresses.
		* Relational databases handle links by forming a key into another table.
	* Second, the way collections are handled.
		* Objects can easily use collections to handle multiple references from a single field.
		* In relational databases, normalization forces all relation links to be single valued.
		* e.g. Order object naturally has links to line items as a collection of references to line item objects and line item does not have links to order, while the relational table structure is other way around, each line item record maintains a link to the order object.

#### Identity Field

* Handle representation problem by keeping the relational identity of each object as an Identity Field in the object, and to look up these values to map back and forth between object references and relational keys.
*   When reading objects from disk use an Identity Map as a look-up table from relational keys to objects.

#### Foreign Key Mapping

* Each time you come across a foreign key in a table use Foreign Key Mapping to wire up appropriate inter object references.
* If the key is not in the Identity Map, go to the database or use Lazy Load
* While saving an object, save it into the row with the right key. Any inter object's reference is replaced with the target object's ID field.

####  Handling objects with collections (complex version of Foreign Key Mapping)

* If an object has a collection
	* Loading
		* Issue another query to find all the rows that link to ID of the source object.
		* Each object that comes back gets added to the collection.
	* Saving
		* Save each object in the collection.
		* Make sure it has a foreign key to the source object.
	* Messy when we need to detect objects, added, edited and removed from the collection.
	* Better to use some form of metadata-based approach.
* Use Dependent Mapping to simplify mapping, if the collection isn't used outside the scope of the source object.

### Association Table Mapping

* Used  to handle many-to-many relationships.
* Create a new relational table to handle the many-to-many association.

#### Common Gotchas

##### Use unordered sets instead of ordered collections

* Don't rely on ordering within a collection.
* In OO languages it is common to use List or Array and it makes testing easier.
* Nevertheless it is difficult to maintain an arbitrary ordered collection when saved to a relational database.
* Consider using unordered sets for storing collections.
* Decide on sort order whenever you do a collection query.

##### Defer referential integrity checks to end of a transaction

* If not, referential integrity checks are done for each update.
* Defer checks to the end, be careful with updates and their order.
* Either do a topological sort on the updates or hard code the order in which tables are written to database.
* This reduces deadlocks in the database that cause transactions to rollback too often.

### Value Objects

* Don't use Identity Map for Value Objects.
* Small value objects like currency and date and time shouldn't be represented as their own table in the database.
* Take the value object and embed them into the linked object as Embedded Value.
* Since Value Objects have value semantics, create them each time with a read.
* While saving, just dereference the object and spit of the fields into the owning table.

> **NOTE** - .NET provides excellent support for value objects using `struct`.

### Serialized LOB

* Taking a while cluster of objects and saving them as a single column in a table as Serialized LOB.
* LOB - Large Objects - Can be BLOB - Binary Large Object or CLOB - Character Large Object.
* One obvious route - serialize an hierarchic object structure as XML.
* In a single read, we can grab the whole bunch of objects.
* Serialized LOB can save lot of database round-trips as databases perform poorly with small highly interconnected objects, where lot of time is spent in making small database calls.
* Obvious downside: SQL cannot make portable queries against the data structure!
* Serialized LOB is best used when you don't need to query parts of the stored structure.
* Don't use this too much as it turns the database into little more than a transactional file system.

> **NOTE** - Document databases (NoSQL) like MongoDB and Cassandra greatly support storing and querying whole object hierarchies or collection of objects as documents.