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
- The server component will be the registry in which all the microservices register their availability. 
- The microservices use the Eureka client to register; once the registration is complete, it notifies the server of its existence. 

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
logging.level.com.netflix.eureka=ON
logging.level.com.netflix.discovery=ON
```
<br>

**Step 3:Add the @EnableEurekaServer annotation in your application Java file**

Below is our `EurekaApplication.java` file - the only difference is the `@EnableEurekaServer` annotation which tells Spring to activate the Eureka server related configuration:

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



