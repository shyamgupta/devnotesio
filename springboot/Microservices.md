---
layout: springboot
permalink: /springboot/Microservices/
---
## Introduction
Microservices are independently deployable services modeled around a business domain.
The Microservices Architecture (MSA) decomposes systems into discrete, individual, standalone components that can communicate amongst themselves, working together or with external systems.

What sets a microservices architecture apart from more traditional, monolithic approaches is how it breaks an app down into its core functions. Each function is called a service, self-contained and can be built and deployed independently, meaning individual services can function (and fail) without negatively affecting the others.

<img src="/assets/img/microservices.png" alt="Microservices" class="img-fluid img-thumbnail">

- **Independent Deployability**: Independent deployability is the idea that we can make a change to a microservice and deploy it into a production environment without having to utilize any other services. To guarantee independent deployability, we need to ensure our services are loosely coupled—in other words, we need to be able to change one service without having to change anything else. This means we need explicit, well-defined, and stable contracts between services.
- **Modeled Around a Business Domain**: Rolling out a feature that requires changes to one or more microservices is expensive. You need to coordinate the work across each service (and potentially across separate teams) and carefully manage the order in which the new versions of these services are deployed. In a traditional 3-Tier archirecture, each layer represents a different service boundary, with each service boundary based on related technical functionality. Experience has shown that changes in functionality typically span multiple layers in these types of architectures—requiring changes in presentation, application, and data tiers. By making our services end-to-end slices of business functionality, as shown below, we ensure that our architecture is arranged to make changes to business functionality as efficient as possible. Each service, if needed, can encapsulate presentation, business logic, and data storage.

<img src="/assets/img/domain.png" alt="Business Domain" class="img-fluid img-thumbnail">

## Service Registry
Microservices will interact with each other, and since they are interacting over the network, they need to know where to find each other. However, since you’ll have many microservices running, you may eventually have to change a port, or even move one of them to a different server.

Having to change the configuration for all other microservices that use this component, just because it moved, isn’t really great. Luckily, there are solutions to that problem, such as a service registry.

Service Registry is collective information of services along with related details including, the location of service and number of instances. Since service registry is the database of services available, it must be highly available. There are two stages of interaction with Service Registry-

- **Registration** — Whenever a new service or service instance scales in/out, it needs to register/de-register itself with Service Registry. There are two flavours of service registration:
	- Self Registration: The services own the responsibility of registering itself with Service Registry on startup and de-register on shutdown. An example implementation of self-registration is using Netflix’s Eureka client to register services itself with Eureka Service Registry. Spring has incorporated Eureka into Spring Cloud, making it easier to stand up a Eureka server which is responsible for registration and discovery of Microservices.
	- Third-Party Registration: Thrid-party registration allows delegation of service registration/de-registration task to Third-party registrar (service manager) component. On service instance start-up, service manager is responsible for registering the service with Service Registry. Similarly, it de-registers on service shutdown.
- **Discovery** — Service discovery is the counter aspect of service registration. For the client to make calls to service instances, it first needs to locate the available service and its details and then proceed with actual call.

## Eureka Server
Every microservice will be registered with the Eureka server. Eureka consists of a server and a client-side component. 
- The server component will be the registry in which all the microservices register their availability. The registration typically includes service identity and its URLs.
- The microservices use the Eureka client to register; once the registration is complete, it notifies the server of its existence. The client is always a part of an application and is responsible for connecting to a remote discovery server. Once the connection is established it should send a registration message with a service name and network location. The consuming components will also use the Eureka Client for discovering the service instances. When a microservice is bootstrapped, it reaches out to the Eureka Server and advertises its existence with the binding information. Once registered, the service endpoint sends ping requests to the registry every 30 seconds to renew its lease. The registry information is replicated to all Eureka clients so that the clients need to go to the remote Eureka Server for each and every request. Eureka clients fetch the registry information from the server and cache it locally. After that, the clients use that information to find other services. When a client wants to contact a microservice endpoint, the Eureka Client provides a list of currently available services based on the requested service ID. The communication between the Eureka Client and Eureka Server uses REST and JSON.

<img src="/assets/img/eureka.png" alt="Eureka Server" class="img-fluid img-thumbnail">

Below are the steps for standing up a Eureka server in Spring Boot

**Step 1:Maven Dependencies**

- **spring-cloud-starter-config**: This initializes and sets up Spring Cloud
- **spring-cloud-netflix-eureka-server**: This auto configures the Eureka server

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-config</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-netflix-eureka-server</artifactId>
</dependency>
```
<br>
**Step 2: Update the application.properties file**

Below is our `application.properties` file where we are configuring our Eureka server name and port. Additionally, we've specified that the server should NOT register itself as a client (i.e. our application will act a Eureka server in this case) and logging have been enabled for Eureka server and discovery:

```properties
spring.application.name=eureka-server
server.port=8761

#dont register itself as a client
eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false
logging.level.com.netflix.eureka=TRACE
logging.level.com.netflix.discovery=TRACE
```
<br>

**Step 3:Add the @EnableEurekaServer annotation in your application Java file**

Below is our `EurekaApplication.java` file - the only difference is the `@EnableEurekaServer` annotation which tells Spring to activate the Eureka server related configuration. The @EnableEurekaServer annotation will start the embedded Eureka server in our application and make it ready to use. It will enable the service registry in our application as well:

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer
public class EurekaApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaApplication.class, args);
    }

}
```
<br>

At this point, you can visit `localhost:8761` to view the Eureka dashboard. Under the **"Instances currently registered with Eureka"** is where we will see Microservices once they are created and registered.

## Sping Data REST: Create REST endpoints

Spring Data REST makes it easy to expose microservices. Spring Data REST builds on top of Spring Data repositories and automatically exports those as REST resources. Below is how Spring Data REST works - there is no need to create a controller or service layer:

1. At application startup, Spring Data Rest finds all of the spring data repositories
2. Then, Spring Data Rest creates an endpoint that matches the entity name
3. Next, Spring Data Rest appends an S to the entity name in the endpoint. For example, if I have an ItemRepository which extends `CrudRepository<Item, Long>`, 
	- Spring Data REST will expose a collection of resources at `http://localhost:8080/items/` - the path is derived from ***un-capitalized, pluralized simple class name of the Entity class*** being managed, in this case Item. 
	- It also exposes `http://localhost:8080/items/{id}`, i.e an item resource for each of the items managed by the repository.
4. Lastly, Spring Data Rest exposes CRUD (Create, Read, Update, and Delete) operations as RESTful APIs over HTTP

**Step 1: Maven Dependency**

Add the below dependency to use Spring Data REST:

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-boot-starter-data-rest</artifactId>
</dependency>
```
<br>

**Step 2: Update application.properties file**

```properties
spring.h2.console.enabled=true
spring.h2.console.path=/h2
spring.datasource.url=jdbc:h2:mem:itemsdb
```
<br>

**Step 3: Create the Entity**

```java
@Entity
public class Item{
	@Id
	@GeneratedValue(strategy=GeneratiionType.IDENTITY)
	private Long id;

	private String name;
	private String description;

	public Item(){}

	public Item(Long id,String name,String description){
		this.id = id;
		this.name=name;
		this.description=description;
	}

	//add getters and setters
}
```
<br>

**Step 4: Create the Repository**

```java
public Interface ItemRepository extends CrudRepository<Item,Long>{

}
```
<br>

At this point, you can visit localhost:8080/items and localhost:8080/items/{id} to view the endpoints exposed by Spring Data REST.

## Convert the Microservice to a Eureka Client and Register it with Eureka Server

We will now convert the Microservice into a Eureka client - doing this will cause the service to act like a Spring discovery client and will register itself with the Eureka server. For a @SpringBootApplication to be discovery-aware, we need to add the below dependencies and use the @EnableEurekaClient annotation:

**Step 1: Maven Dependency**

Add the below dependencies:
- **spring-cloud-starter-netflix-eureka-client**: This will make the service register itself with the Eureka server.
- **spring-cloud-starter-config** 
- **spring-cloud-starter-parent**

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-config</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-parent</artifactId>
	<version>Greenwich.RELEASE</version>
	<type>pom</type>
	<type>import</type>
</dependency>
```
<br>

**Step 2: Add the @EnableEurekaClient annotation**
Annotate the main Spring application class with the @EnableEurekaClient annotation. @EnableEurekaClient is optional if the spring-cloud-starter-netflix-eureka-client dependency is on the classpath. 

```java
@SpringApplication
@EnableEurekaClient
public class ItemsApplication{
	public static void main(String args[]){
		...
	}
}
```
<br>

**Step 3: Update application.properties File**

spring.application.name uniquely identifies the client among the list of registered clients.

```properties
spring.application.name=item-service
server.port=8762
eureka.client.serviceUrl.defaultZone=http://localhost:8761/eureka/
eureka.client.service-url.default-zone=http://localhost:8761/eureka/
eureka.instance.prefer-ip-address=true
```
<br>

At this point, when you run the Microservice, you should see the microservice listed in the Eureka dashboard. You can also visit http://localhost:8762:/items to view the endpoint.



