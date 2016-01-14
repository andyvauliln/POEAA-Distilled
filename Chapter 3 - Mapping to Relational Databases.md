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
> **NOTE** - .NET provides an excellent support for value objects using `struct`.

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

