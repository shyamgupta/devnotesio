---
layout: springboot
permalink: /springboot/JPA/
---
## Persistence in Java: JPA
The Java Persistence API (JPA) specification and it's implementations help developers in accessing, persisting and managing data between Java objects, classes and relational databases. Object Relational Mapping (ORM) connect Java and relational db's. 

Relational databases store data in rows and columns, but Object-Oriented languages like Java does not know about rows and columns. Object Oriented languages organize data in objects, classes,packages, interfaces etc. and relational db's know nothing about these Object Oriented concepts. ORM bridges this gap bewteen Object Oriented Languages and Relational Databases - it allows objects to be mapped to tables and tables can be mapped to objects. 

The process of ORM is often delegated to external tools or frameworks. In Java EE this framework is called JPA, which is a specification for accessing, persisting and managing data between Java objects, classes and relational databases. 

Using JPA, we can access a database without writing any SQL code. JPA is an abstraction over JDBC (Java Database Connectivity). 

## Hibernate

JPA lives in the `javax.persistence` package, with different sub-packages, but most of the core API and annotations live in javax.persistence. Since JPA is a specification (set of empty methods and collection of interfaces) or a set of guidelines, it cannot do anything on its own. In order to be fullly functional, JPA needs an implementation, also known as an instance provider. Some of the JPA implementations are:

1. EclipseLink (Oracle)
2. Hibernate (JBOSS and RedHat): Spring with JPA uses Hibernate as the persistence provider.
3. OpenJPA (IBM)
4. Data Nucleus (JPOX)

## JPA Metadata

The mapping between Java objects and database tables is defined via the persistence metadata, which is used by the JPA instance provider to perform correct database operations. There are two ways to define JPA metadata:

1. **Annotations**: Using annotations in the Java class, @Entity, @Id, @Table, @Column etc. is the most common way and is what we will use. Spring can scan all packages looking for entities described by the @Entity annotation.
2. **XML**: Alternatively, metadata can be defined in XML or a combination of XML and Annotations. 

JPA can be setup with no XML, which is very convenient.

## Entities

Entities are objects that live in a database and have the ability to be mapped to a database. They are defined by the `@Entity` annotation, and support inheritence, relationships etc and also have a unique identifier. 

```java
@Entity
@Table(name="applications")
public class Application{

	@Id
	@GeneratedValue(strategy=GenerationType.AuTO)
	private Integer id;

	@Column(name="app_name",nullable=false)
	private String name;

	@Column(length=200)
	private String description;
	private String owner;
}
```
<br>

- **@Entity** annotation defines that a class can be mapped to a database table.
- **@Table** defines the table that the entity is mapped to.
- **@Id** marks a field as a primary key field.
- **@GeneratedValue** generates a unique identifier for the entity.
- **@Column** changes the name or length of any column in the database.

Once the entity has the correct mapping metadata, JPA allows us to query the entity in an object-oriented way. The entity manager, `javax.persistence.EntityManager` is responsible for orchestrating this process - it performs CRUD operatations, allowing is to persist, update, retrieve, and delete objects from a database. Enttity Manager API allows us to interact with the database without writing any JDBC or SQL code.

JPA also comes with a more powerful language, **Java Persistence Query Language (JPQL)** - allowing us to retrieve data with an object-oriented query language. JPQL is a layer on top of SQL. 

1. Note that ***results from a JPQL query are returned as a collection of entities***.
2. JPQL does NOT support schemaless or NoSQL databases. It also does not support JSON.




























