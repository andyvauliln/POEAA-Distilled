# Chapter 3 - Mapping to Realtional Databases
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
* NoSQL and Polyglot Peristence are techniques used nowadays, which in a way is nothing but OODBs.
* NoSQL is gaining traction and more support. Earlier Object Databases has little support / adoption as compared to RDBMS.
* Implementations include - See [Wikipedia](https://en.wikipedia.org/wiki/Object_database#External_links)
    * Flat file
    * Column-oriented
    * Document-oriented
    * Object-relational
    * Deductive
    * Temporal
    * XML data stores
    * Triplestores

### ORM - Object Relational Mapper
* Even if you cannot consider using OODB consider buying a ORM tool.
* Builing a Data Mapper is a complicated endeavor
* These tools aren't cheap - Make the decision between buy Vs. build!
* Even if you buy these ORM tools, it is good to know these patterns, as it might help in tuning a tool as it takes a small but significant chunk of work.