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
