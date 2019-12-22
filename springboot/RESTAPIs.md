---
layout: springboot
permalink: /springboot/RESTAPIs/
---

## Introduction

Web Services, API's and Microservices allow you to share data between two separate systems. Web Services are designed to communicate with other programs or applications (instead of end users).

- **Web Services**: Web Services are designed to communicate with other programs or applications (instead of end users). Complexities of 1990's SOAP based Web Services led to REST APIs. All Web Services are API's but not vice versa.
- **APIs**: APIs allow for data sharing between two systems. However, API's are more dynamic, lightweight architecture, and streamlined. API's are good for devices with limited bandwidth (like cellphones).
- **Microservices**: Similar to API's but are fully contained - they are individual components that communicate with each other, and are modeled around a specific business domain.

## Web Services

A web service is a way to share data between two disparate systems or a way to retrieve data to display in your application. Web Services are designed to communicate with other programs or applications (instead of end users). The communication typically happens between a client and a server. 

- Client - The client makes a request for data.
- Server (a.k.a. Service Provider) - The server responds to the client's request.

The means of communication between the client and server is via a standard web protocol like HTTP (or HTTPS) and uses a common language like JSON or XML. A client invokes a web service by sending an XML (or JSON) message, then waits for a corresponding XML response from the server.

1. The service provider defines a standard format for requests (XML or JSON) and also for the response provided.
2. The client makes a request of the web service across the network
3. The web service receives that request, performs some action, and sends the response back to the client.

Benefits of a Web Service:
- Code Reusability: Web Services are usually small components that can be used by multiple systems. Web Services make sense when we want to use the same data in several applications. For example, displaying the LinkedIn feed on cellphone and laptop could use the same Web Service.
- Usability: Web Services are an easy way to expose business logic and data to other systems in a secure way to a wide range of audiences and platforms.

## REST Architecture

REST stands for **RE**presentational **S**tate **T**ransfer - it's a set of guidelines used for developing API's. There are 4 principles API's follow:

1. Data and functionality in the API are considered as resources and identified through something called the URI, or Uniform Resource Identifier. These are accessed by web links.
2. Resources are manipulated using a fixed set of operations - GET, POST, PUT, DELETE
3. Resources can be represented in multiple formats (HTML, plain text, XML etc)
4. Communication  between the client and the endpoint is stateless.

We will see how we can develop REST API's using Spring Boot. Our API is about a dog database stored in an in-memory H2 database.

## Maven Dependencies
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

We will need to update our `application.properties` file to indicate the location of H2 console

```
spring.h2.console.enabled=true
spring.h2.console.path=/h2

spring.datasource.url=jdbc:h2:mem:dogdata
```
<br>
## data.sql

Place this file under `/src/main/resources`, same level as `application.properties`. This is our sample database for the application.

```sql
INSERT INTO dog (id, name, breed, origin) VALUES (1, 'Fluffy', 'Pomeranian', 'Mountain View, CA');
INSERT INTO dog (id, name, breed, origin) VALUES (2, 'Spot', 'Pit Bull', 'Austin, TX');
INSERT INTO dog (id, name, breed, origin) VALUES (3, 'Ginger', 'Cocker Spaniel', 'Kansas City, KS');
INSERT INTO dog (id, name, breed, origin) VALUES (4, 'Lady', 'Direwolf', 'The North');
INSERT INTO dog (id, name, breed, origin) VALUES (5, 'Sasha', 'Husky', 'Buffalo, NY');
```
<br>
## Entity
This is where we will be modeling the database as a Java class:

```java
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Entity
public class Dog {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
	private String name;
	private String breed;
	private String origin;
	public Long getId() {return id;}

	public void setId(Long id) {this.id = id;}

	public String getName() {return name;}

	public void setName(String name) {this.name = name;}

	public String getBreed() {return breed;}

	public void setBreed(String breed) {this.breed = breed;}

	public String getOrigin() {return origin;}

	public void setOrigin(String origin) {this.origin = origin;}
	public Dog(Long id, String name,String breed,String origin) {
		this.id = id;
		this.name = name;
		this.breed = breed;
		this.origin = origin;
	}
	public Dog(String name, String breed) {
		this.name = name;
		this.breed = breed;
	}
	public Dog() {}
}
```
<br>

## Repository
Repository interface will execute the actual SQL queries and interact with the database to perform CRUD operations:

```java
import java.util.List;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.CrudRepository;
import com.udacity.entity.Dog;

public interface DogRepository extends CrudRepository<Dog, Long> {

	@Query("select d.id,d.breed from Dog d")
	List<String> findAllBreed();
	@Query("select d.id,d.breed from Dog d where d.id=:id")
	String findBreedById(Long id);
	@Query("select d.id,d.name from Dog d")
	List<String> findAllName();
}
```
<br>
## Service Layer
Service interface defines what operations can be performed on our API:

```java
import java.util.List;

import com.udacity.entity.Dog;

public interface DogService {
	List<Dog> retrieveDogs();
	List<String> retrieveDogBreed();
	String retrieveDogBreedById(Long id);
	List<String> retrieveDogNames();
}
```
<br>
In the below class, we are implementing the above interface:

```java
import java.util.List;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import com.udacity.entity.Dog;
import com.udacity.repository.DogRepository;

@Service
public class DogServiceImpl implements DogService {
	@Autowired
	DogRepository dogRepository;
	public List<Dog> retrieveDogs() {
		return (List<Dog>) dogRepository.findAll();
	};
	public List<String> retrieveDogBreed(){
		return (List<String>) dogRepository.findAllBreed();
	};
	public String retrieveDogBreedById(Long id) {
		return (String) dogRepository.findBreedById(id);
	};
	public List<String> retrieveDogNames() {
		return (List<String>) dogRepository.findAllName();
	};
}
```
<br>
## Web

In our controller:
- **ResponseEntity**: It is used to set the body, status, and headers of an HTTP response. If we want to use it, we have to return it from the endpoint; Spring takes care of the rest. ResponseEntity is a generic type. As a result, we can use any type as the response body.

```java
import java.util.List;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;
import com.udacity.entity.Dog;
import com.udacity.service.DogService;

@RestController
public class DogController {
	private DogService dogService;
	
	@Autowired
	public void setDogService(DogService dogService) {
		this.dogService = dogService;
	}
	@GetMapping("/dogs")
	public ResponseEntity<List<Dog>> getAllDogs() {
		List<Dog> list = dogService.retrieveDogs();
		return new ResponseEntity<List<Dog>>(list, HttpStatus.OK);
	}
	@GetMapping("/dogs/breed")
	public ResponseEntity<List<String>> getDogBreeds() {
		List<String> list = dogService.retrieveDogBreed();
		return new ResponseEntity<List<String>>(list, HttpStatus.OK);
	}
	@GetMapping("/{id}/breed")
	public ResponseEntity<String> getBreedByID(@PathVariable Long id) {
		String breed = dogService.retrieveDogBreedById(id);
		return new ResponseEntity<String>(breed, HttpStatus.OK);
	}
	@GetMapping("/dogs/name")
	public ResponseEntity<List<String>> getDogNames() {
		List<String> list = dogService.retrieveDogNames();
		return new ResponseEntity<List<String>>(list, HttpStatus.OK);
	}
}
```


