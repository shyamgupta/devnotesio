---
layout: springboot
permalink: /springboot/SpringData/
---
## Introduction
Spring Data comes with a common programming model for persisting data in various types of database engine, ranging from traditional relational databases (SQL databases) to various types of NoSQL database engine, such as document databases (for example, MongoDB), key-value databases (for example, Redis), and graph databases (for example, Neo4J).

The two core concepts of the programming model in Spring Data are entities and repositories. Entities and repositories generalize how data is stored and accessed from the various types of database.

- **Entity**: An entity describes the data that will be stored by Spring Data. Entity classes are, in general, annotated with a mix of generic Spring Data annotations and annotations that are specific to each database technology. For example, an entity that will be stored in a relational database can be annotated with JPA annotations such as @Entity.  Each entity class generally represents a single database table, and an instance of such a class contains the data of a single table row. Each entity instance always has a unique object identifier, which is the same thing to an entity that a primary key is to a database table.
- **Repository**: Repositories are used to store and access data from different types of database. In its most basic form, a repository can be declared as a Java interface, `Repository<T,ID>` where T is the type being stored and ID is a primary key type -  Spring Data will generate its implementation on the fly.  Spring Data also comes with some base Java interfaces, for example, `CrudRepository`, to make the definition of a repository even simpler. The base interface, CrudRepository, provides us with standard methods for create, read, update, and delete operations. If we want to use the repository, we can simply inject it (using @Autowired) and then start to use it. The actual strength of Spring Data JPA lies in the repository abstraction. It abstracts the data store specific implementation details and allows us to write the business logic code on a higher abstraction level. We only need to learn how to use the Spring Data repository interfaces instead of worrying about the implementation of these interfaces. Spring Data provides an implementation for these repository interfaces out-of-the-box.

## Object-Relational Mapping (ORM) and Java Persistence API (JPA)
Object-Relational Mapping (ORM) is a technique that allows you to fetch and manipulate from a database by using an object-oriented programming paradigm. ORM is really nice for programmers because it relies on object-oriented concepts, not on database structure. It also makes development much faster and reduces the amount of source code. ORM is mostly independent of the databases and developers don't have to worry about vendor-specific SQL statements.

Java Persistent API (JPA) provides object-relational mapping for Java developers. The JPA entity is a Java class that presents the structure of a database table. The fields of an entity class present the columns of the database tables.

Hibernate is the most popular Java-based JPA implementation, and it is used in Spring Boot as a default. It is a mature product and it is widely used in large-scale applications.

## Layers of a Persistence Tier
The application tier that deals with persistence is often called the persistence tier. Spring helps to enforce a modular architecture in which the persistence tier is divided into several core layers that contain the following:
- **Domain Model**: The domain model ***represents the key entities within an application***, defining the manner in which they relate to one another. It is simply a Java Bean that represents our "business data". Each entity defines a series of properties, which designates its characteristics, as well as its relationships to other entities. Each class within the domain model contains the various properties and associations that correlate to columns and relationships within the database. Part of Hibernate’s job is to convert between domain model instances and rows in the database. Since this class must be a Java Bean, it must have private attributes, getter and setters in the getField() and setField() formats, and an empty constructor.
- **Data Access Object (DAO) layer**:  DAO is a design pattern in which a data access object (DAO) is an object that provides an abstract interface to some type of database or other persistence mechanisms. ***The DAO layer defines the means for CRUD operations on the domain model data***. The goal of the DAO pattern is to completely abstract the underlying persistence technology and the manner in which it loads, saves, and manipulates the data represented by the domain model. For example, if I have a Person domain entity, we might create a PersonDao class to define all the application’s persistence needs related to the Person entity- we would likely have a methods like findById(Long id) for loading a Person entity from the database using its unique identifier. When defining a DAO, it is good practice to first write the interface and it's recommended to create separate DAOs for each persistent entity in your domain model.
- **Service Layer**: The layer that ***handles the application business logic*** is called the service layer. The service layer typically defines an application’s public-facing API. Service layer provides the logic to operate on the data sent to and from the DAO and the client. If the DAO layer manages the persistence of data, the service layer, on the other hand, exposes all DAO transactions through its own set of interfaces and implementations.

<blockquote class="blockquote">
	In Spring MVC, the business logic is handled by the Service Layer and the data is handled by the Persistence Layer (domain model + repositories). The Service Layer and Persistence Layer together make up for what is known as the M in MVC in today's web applications.
</blockquote>

<img src="/assets/img/mvc.png" alt="Spring MVC" class="img-fluid img-thumbnail">


## Entity: Mapping a Database Table to a Java Class
Let's start with a simple example, where we map one database table to one JPA entity: below we have a STUDENT table with 4 columns:

```sql
student_id INT(11)
student_name VARCHAR(45)
student_fulltime TINYINT(1)
student_age INT(11)
```
<br>

Below we have a Java Student class with 4 attributes that represents the above database table. 
- The class must be annotated with `@Entity`. Adding a JPA @Entity and @Id annotation to a POJO makes it a persistable object.
- Entities must have a unique identfier annotated with `@Id`. The id field is used to hold the database identity of each stored entity—the primary key when using a relational database. Here, we are delegating the responsibility to generate unique values of the identity field to Spring Data. The primary key is defined by using the @Id annotation. The @GeneratedValue annotation defines that the ID is automatically generated by the database. We can also define our key generation strategy. Type AUTO means that the JPA provider selects the best strategy for a particular database. We don't need an ID field in our constructor due to automatic ID generation. The JPA specification supports 4 different primary key generation strategies which generate the primary key values programmatically or use database features:
	- ***GenerationType.AUTO*** : This is the default generation type and lets the persistence provider choose the generation strategy. In this case, the primary key values will be unique at the database level.
	- ***GenerationType.IDENTITY*** :  Here the persistence provider must assign primary keys for the entity using a database identity column. IDENTITY column is typically used in SQL Server. This special type column is populated internally by the table itself without using a separate sequence. 
	- ***GenerationType.SEQUENCE*** : It uses a database sequence to generate unique values. This generator uses sequences if they're supported by our database, and switches to table generation if they aren't.
	- ***GenerationType.TABLE***: Gets only rarely used nowadays. It simulates a sequence by storing and updating its current value in a database table. 

- `@Column` maps the class atributes to table columns. The database columns are named according to class field naming by default. If you want to use some other naming convention, you can use the @Column annotation. With the @Column annotation, you can also define the column's length and whether the column is nullable.
- **Every persistent class must have a zero-argument (empty) constructor**. You can either define this constructor yourself, or not write any constructors at all, which produces a default empty constructor.
- Each persistent class may have properties that represent the data which the class holds, defined as simple values, or other persistent classes, or both. These properties are usually defined as nonpublic instance variables, and are accessed through getters and setters which follow a specific naming convention. For example, for the Student class with the firstName property of type String, the getter and setter methods are respectively named getFirstName() and setFirstName(). Notice the lower-case f in the property name, and the upper-case F in the method names. The exception to this naming convention is Boolean properties, which uses a method called isXxx to look up their values. For example, the Student class might have methods called isGraduated() (which takes no arguments and returns a Boolean) and setGraduated() (which takes a Boolean and has a void return type), and would be said to have a Boolean property named graduated. (Again, notice the lower-case first letter in the property name.) 

JPA creates a database table called the name of the class when the application is initialized. If you want to use some other name for the database table, you can use the @Table annotation.

```java
@Entity
@Table(name="STUDENT")
public class Student{
	@Id
	@GeneratedValue(strategy=GenerationType.AUTO)
	@Column(name="student_id")
	private Integer studentId;

	@Column(name="student_name")
	private String name

	@Column(name="student_fulltime")
	private boolean fullTime;

	@Column(name="student_age")
	private Integer age;

	public Student(){}

	public Student (String name, boolean fullTime, Integer age){
		this.name = name;
		this.fullTime = fullTime;
		this.age = age;
	}

	//getters and setters logic here
}
```
<br>

Now that we have define dthe object to relational mapping metadata, all of our database related coding can stay in the logical world because JPA will take care of the physical world for us.

## Repository

The Spring Boot Data JPA provides a CrudRepository interface for CRUD operations. It provides CRUD functionalities to our entity class. ***A repository is a "data access" pattern that contains methods for performing CRUD operations***. We do this by creating a Java interface that provides CRUD operations for Student objects by extending the `CrudRepository<Class_Name, ID_Type>` interface:

```java
@Repository
public interface StudentRepository extends CrudRepository<Student, Integer> {}
```
<br>

The preceding interface might appear empty, but the inherited methods inside CrudRepository provide the core CRUD operations we need. The interface is generically typed to match up with the domain class and the ID's field type.

With this code in place, when Spring Boot creates an application context, Spring Data JPA will scan and discover our repository definition. Then it will automatically generate a concrete proxy that implements this interface. This saves us the labor of writing all these queries. Below are some useful methods that CrudRepository includes:

By inheriting from CrudRepository, we can call many methods without the need to implement them ourself. We can also define our own methods. These method names should use special keywords such as “find”, “order” with the name of the variables. Spring Data JPA developers have tried to take into account the majority of possible options that you might need. In our example for the Student entity, `findByStudentName(String student_name)` method returns all entries from the table with the specified student_name.

**Create Method**
- Class_Name save(Class_Name object_variable)

**Read Methods**
- Optional<Class_Name> findById(ID id)
- boolean existsById(ID id)
- Iterable<Class_Name> findAll()
- findOne(ID id)
- Iterable<Class_Name> findAllById(Iterable<ID> iterable)
- long count()

**Update Method**
- Iterable<Class_Name> saveAll(Iterable<Class_Name> iterable)

**Delete Methods**
- void deleteById(ID id)
- void delete (Class_Name object_Name)
- void deleteAll(Iterable<Class_Name> iterable)
- void deleteAll()

## Service Layer
Controllers are supposed to be "thin" and shouldn’t contain any real business logic. The controller's job is to simply handle the request and move the user into the correct view or return the response in an API call. We could have accessed the Repository from the Controller. However, as a best practice, it is much cleaner to have the Service Layer perform all of the access to the database. This practice refers to the Separation of Concerns pattern. The benefits a Service Layer provides is that it defines a common set of application operations available to different clients and coordinates the response in each operation.

## REST API
- REST is an architectural style for the web, a set of design criteria.
- REST applications are defined based on the resource they represent.
- A resource can be a document, a row in a database table etc
- REST API's are defined around URIs (if there is no URI then we cannot talk about a resource)
- Best practice is to have sub URIs as query parameters.
- As per the expected REST API guidelines, creation of a new resource should post data to the root of our endpoint.

To understand what REST is all about, it helps to break down the acronym into its constituent parts:

- Representational: REST resources can be represented in virtually any form, including XML, JavaScript Object Notation (JSON), or even HTML—whatever form best suits the consumer of those resources.
- State: When working with REST, you’re more concerned with the state of a resource than with the actions you can take against resources.
- Transfer: REST involves transferring resource data, in some representational form, from one application to another.

Resources in REST are identified and located with URIs. There are no strict rules regarding RESTful URI structure, but the  URI should identify a resource, not bark a command to the server. Again, the focus is on things, not actions.       

For example, a blog post can be identified by the URI http://www.myblog.com/posts/restful-architecture . A resource can be a collection resource , which represents a grouped set of resources. For example, the URI http://www.myblog.com/posts/ represents the posts resource, which may contain zero or more Post resources, each of which can be identified by its own URI.

REST is neither a specification nor a standard. It's a way of representing data and manipulating a resource that resides on the server.

## Example
Let's see all of the above concepts in action - our REST API will consist of Dog data stored in an in-memory H2 database and will have the below REST endpoints:

<table class="table table-striped table-bordered table-hover table-responsive-sm">
	<thead class="bg-danger text-light">
		<tr>
			<th>HTTP Verb</th>
			<th>REST Endpoint</th>
			<th>Description</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>GET</td>
			<td><code>/api/dogs/</code></td>
			<td>Returns a list of all dogs in the database</td>
		</tr>
		<tr>
			<td>GET</td>
			<td><code>/api/dogs/{id}</code></td>
			<td>Returns the dog with the specified id</td>
		</tr>
		<tr>
			<td>POST</td>
			<td><code>/api/dogs/</code></td>
			<td>Create a new entry (dog) in the database</td>
		</tr>
		<tr>
			<td>PUT</td>
			<td><code>/api/dogs/{id}</code></td>
			<td>Update an existing entry (dog) in the database</td>
		</tr>
		<tr>
			<td>DELETE</td>
			<td><code>/api/dogs/{id}</code></td>
			<td>Delete an entry (dog) from the database</td>
		</tr>
	</tbody>
</table >

#### Dependencies

We've added Spring Web, JPA and H2 dependencies:

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
	<groupId>com.h2database</groupId>
	<artifactId>h2</artifactId>
	<scope>runtime</scope>
</dependency>
```
<br>

#### H2 database
In our `application.properties` file, we've enabled H2 database console and the database location:

```
spring.h2.console.enabled=true
spring.h2.console.path=/h2
spring.datasource.url=jdbc:h2:mem:dogdb
```
<br>

We've created a `data.sql` file under `src/main/resources`:

```sql
INSERT INTO dog (id, name, breed, origin) VALUES (1, 'Fluffy', 'Pomeranian', 'Mountain View, CA');
INSERT INTO dog (id, name, breed, origin) VALUES (2, 'Spot', 'Pit Bull', 'Austin, TX');
INSERT INTO dog (id, name, breed, origin) VALUES (3, 'Ginger', 'Cocker Spaniel', 'Kansas City, KS');
INSERT INTO dog (id, name, breed, origin) VALUES (4, 'Lady', 'Direwolf', 'The North');
INSERT INTO dog (id, name, breed, origin) VALUES (5, 'Sasha', 'Husky', 'Buffalo, NY');
```
<br>

#### Entity

```java
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Entity
public class Dog {
	
	@Id
	@GeneratedValue(strategy=GenerationType.IDENTITY)
	private Long id;
	
	private String name;
	private String breed;
	private String origin;
	
	//Constructors
	public Dog() {}

	public Dog(String name, String breed, String origin) {
		super();
		this.name = name;
		this.breed = breed;
		this.origin = origin;
	}
	
	//Getters and Setters

	public Long getId() {
		return id;
	}

	public void setId(Long id) {
		this.id = id;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public String getBreed() {
		return breed;
	}

	public void setBreed(String breed) {
		this.breed = breed;
	}

	public String getOrigin() {
		return origin;
	}

	public void setOrigin(String origin) {
		this.origin = origin;
	};
	
	
}
```
<br>

#### Repository
```java
import org.springframework.data.repository.CrudRepository;
import org.springframework.stereotype.Repository;

import com.shyamgupta.entity.Dog;

@Repository
public interface DogRepository extends CrudRepository<Dog,Long>{

}
```
<br>

#### Service Layer
Below is the Service interface.
Note, we are making use of Java's `Optional` class which was introduced in Java 8. It is a class that encapsulates an optional value. You can view Optional as a single-value container that either contains a value or doesn't (it is then said to be "empty"). Its purpose is to help design more-comprehensible APIs so that by just reading the signature of a method, you can tell whether you can expect an optional value. This forces you to actively unwrap an Optional to deal with the absence of a value. Below are some of the useful methods in this class we are making use of:
- `isPresent()`: Checks if there is a value inside the Optional object. A value is present only if we have created Optional with a non-null value. This method returns true if the wrapped value is not null.
- `get()`: Retrieves the wrapped value only if the wrapped object is not null.

```java
import java.util.List;
import java.util.Optional;

import com.shyamgupta.entity.Dog;

public interface DogService {
	
	//CREATE
	Dog createDog(Dog d);
	
	//READ
	List<Dog> getAllDogs();
	Optional <Dog> getDog(Long id);
	
	//UPDATE
	Dog updateDog(Dog d, Long id);
	
	//DELETE
	boolean deleteDog(Long id);

}
```
<br>

Below is the implementation of the Service interface:

```java
import java.util.List;
import java.util.Optional;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import com.shyamgupta.entity.Dog;
import com.shyamgupta.repository.DogRepository;

@Service
public class DogServiceImpl implements DogService{
	
	@Autowired
	private DogRepository dogRepository;
	
	//GET - Get list of all dogs in the database
	public List<Dog> getAllDogs(){
		return (List<Dog>) dogRepository.findAll();
	};
	
	//GET - Get dog by ID
	public Optional <Dog> getDog(Long id){
		Optional <Dog> dogFound = dogRepository.findById(id);
		if(dogFound.isPresent()) {
			return dogFound;
		}else {
			return null;
		}
		
	}
	
	//POST - Create a new entry in the database
	public Dog createDog(Dog d) {
		return dogRepository.save(d);
	}
	
	//PUT - Update an existing entry in the database
	public Dog updateDog(Dog dogToUpdate, Long id) {
		Optional <Dog> dogFound = dogRepository.findById(id);
		if(dogFound.isPresent()) {
			Dog dog = dogFound.get();
			dog.setName(dogToUpdate.getName());
			dog.setBreed(dogToUpdate.getBreed());
			dog.setOrigin(dogToUpdate.getOrigin());
			dogRepository.save(dog);
			return dog;
			
		}else {
			return null;
		}
	}
	
	//DELETE - Delete an entry from the database
	public boolean deleteDog(Long id) {
		Optional <Dog> dogFound = dogRepository.findById(id);
		if(dogFound.isPresent()) {
			dogRepository.deleteById(id);
			return true;
		}else {
			return false;
		}
	}

}
```
<br>

#### Controller
Our controller will have an entry point for our REST API - to achieve that, we will have a class level request mapping as shown <code>/api/dogs</code>. The path of this entry point should be descriptive as per the REST API guidelines. Since this is an API, our path starts with /api, followed by defining the resource that our controller controls, which is the dog entities in this case. 

```java
import java.util.List;
import java.util.Optional;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.PutMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import com.shyamgupta.entity.Dog;
import com.shyamgupta.service.DogService;

@RestController
@RequestMapping(path="/api/dogs/")
public class DogController {
	
	
	private DogService dogService;
	
	@Autowired
	public void setDogService(DogService dogService) {
		this.dogService = dogService;
	}
	
	//CREATE a new Dog entry in the database
	@PostMapping("")
	public Dog createDog(@RequestParam(value="name") String name,@RequestParam(value="breed") String breed,@RequestParam(value="origin")String origin) {
		Dog newDog = new Dog(name,breed, origin);
		return dogService.createDog(newDog);
	}
	
	//READ - Get list of all dogs
	@GetMapping("")
	public List<Dog> retrieveAllDogs(){
		return dogService.getAllDogs();
	}
	
	//READ -Get Dog by ID
	@GetMapping("{id}")
	public Optional<Dog> getDogById(@PathVariable Long id){
		return dogService.getDog(id);
	}
	
	//UPDATE - Update an existing Dog
	@PutMapping("{id}")
	public Dog updateDog(@PathVariable Long id,@RequestParam(value="name") String name,@RequestParam(value="breed") String breed,@RequestParam(value="origin")String origin) {
		Dog dog = new Dog(name, breed, origin);
		return dogService.updateDog(dog, id);
	}
	
	
	//DELETE an entry from the database
	@DeleteMapping("{id}")
	public boolean deleteEntry(@PathVariable Long id){
		return dogService.deleteDog(id);
	}
}
```
<br>










