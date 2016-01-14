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