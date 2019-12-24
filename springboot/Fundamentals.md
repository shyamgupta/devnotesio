---
layout: springboot
permalink: /springboot/Fundamentals/
---
## Classpath and JAR Files

The concept of a path should be familiar to anyone who has worked on a DOS or Unix platform. It’s an environment variable that provides an application with a list of places to look for some resource. The most common example is a path for executable programs. The Java CLASSPATH environment variable, similarly, is a list of locations that can be searched for packages containing Java class files. Both the Java interpreter and the Java compiler use CLASSPATH when searching for packages and classes on the local host. A location on the class path can be a directory name or the name of a class archive file.

A JAR (**J**ava **Ar**chive) file is a bundle of compiled Java code.  Java supports archives of class files in its own Java archive ( JAR) format, and in the conventional ZIP format. JAR and ZIP are really the same format, but JAR archives include extra files that describe each archive’s contents. JAR files are created with the SDK’s jar utility. The archive format enables large groups of classes to be distributed in a single file; the Java interpreter automatically extracts individual class files from an archive, as needed. Once the JAR file is downloaded, all we need to do is to add it to the classpath - we can then start interacting with the classes and objects defined in the JAR.

## Before Spring

Before the Spring revolution, enterprise applications were generally written using the J2EE standards. In theory, you could deploy your application on any J2EE application server, on any platform. Running your application on an application server has several benefits, the application server offers you services, such as transaction management, messaging, mailing, a directory interface etc. Any J2EE compliant code can make use of these services, as long as the code is written against the interfaces defined in the J2EE specifications.

Unfortunately, there were a few problems with these standards. First of all, the usage of these standards was too complex. Writing a component (EJB: Enterprise Java Bean) required you to write a set of xml files (deployment descriptors), home interfaces, remote/local interfaces, etc. Even worse, 50% of the deployment descriptors were vendor specific, so ‘transparently migrating an application from vendor A to B’ was suddenly not so transparent anymore.

Secondly, there was the ‘look-up’ problem. When a component requires another component, the components itself was responsible for looking up the components it depends upon. Unfortunately, this look-up happens by name, so the name of the dependency was hardcoded in your component.

Last but not least, in a lot of cases, some components did not need all the services the application server provided, but since there was no other API to build components, all your components became heavy weight, bloating your application.

## The POJO Revolution

Coding against the J2EE standards was very cumbersome, you needed to comply to several seemingly arbitrary rules, and it forced you to write code that was not so Object Oriented as one would want to. More and more developers wanted to write just Plain Old Java Objects (POJO’s), without the J2EE standards overhead.

Spring allows you to do programming with very simple non-Spring classes, which means there is no need to implement Spring-specific classes or interfaces, so all classes in the Spring-based application are simply POJOs. That means you can compile and run these files without dependency on Spring libraries.

```java
public class HelloWorld{
	public String hello(){
		return "Hello World";
	}
}
```
<br>
The preceding class is a simple POJO class with no special indication or implementation related to the framework to make it a Spring component. So this class could function equally well in a Spring application as it could in a non-Spring application.

The only problem with POJO’s is that with J2EE (as it existed then) you cannot benefit from the services provided by the container, such as transaction management, remoting, etc. This is where the Spring Framework comes in. This framework brings a lightweight container where your POJO’s live in. You don’t have to support services you don’t need, this also means no unneeded configuration. You don’t have to adhere to some programming model, you can just use POJO’s and declaratively specify what services you want to use.

## Spring Framework

Spring is a framework that integrates all kinds of Java technologies/API’s and makes it possible to use them with simple POJO’s. It provides a nice and elegant way to use existing technologies (such as EJB, Hibernate, JDO, Toplink, JMS, etc). This is accomplished by several support classes and ‘templates’.

For each supported technology there is a module which consists of helper classes to help you implement a certain layer or aspect of your application. The core of Spring, upon which all other modules depend, is the Inversion of Control and Aspect-Oriented programming module. It is these two programming models which are the driving force behind Spring.

Spring's core idea is that instead of managing object relationships yourself, you offload them to the framework. Inversion of control (IOC) is the methodology used to manage object relationships. Dependency injection is the mechanism for implementing IOC.

#### Inversion of Control (IoC) and Dependency Injection (DI)
When developing an application, you always have dependencies between and on components, services, classes etc. Without Inversion of Control you would ‘wire’ these together on the spot where you would need the dependency. The disadvantage of this is that when you would like to use a different implementation of your dependency, you are forced to change your code. With Spring, wiring of these dependencies is taken out of the code, and an external party manages the wiring, namely the container. Hence the name inversion of control, you let something from the outside control how your dependencies are wired together. We speak of dependency injection because the container ‘injects’ the necessary dependencies instead of letting the developer manage them.

Spring provides an object factory - our application can talk to Spring requesting for a new object. Based on the configuration file or annotation, Spring will give us the appropriate implementation. This makes our application configurable.

IoC does just what its name says: it inverts the traditional hierarchy of control for fulfilling object relationships. Instead of relying on application code to define how objects relate to each other, relationships are defined by the framework. Dependency injection (DI) is a mechanism where the framework "injects" dependencies into your app. It's the practical implementation of IOC. 

Let's see how this works in plain old Java: If you're modeling in plain old Java, you might have an interface member on the Car class to reference an Engine interface, as shown below:

```java
public Interface Engine() { ... }

public class Car {
  private Engine engine;
  public Engine getEngine() { ... }
  public void setEngine(Engine engine) { ... }
}
```
<br>

Here we see an interface for an Engine type, and a class for the concrete Car type, which references the Engine. (Note that in a real programming scenario these would be in separate files.) Now, when you're creating a Car instance, you'd set the association as shown below:

```java
// ...
Car newCar = new Car();
Engine sixCylEngine = new InlineSixCylinderEngine();
newCar.setEngine(sixCylEngine );
// Do stuff with the car
```
<br>
Note that you create the Car object first. You then create a new object that fulfills the Engine interface, and assign it manually to the Car object. That is how object associations work in plain old Java.

Now let's look at the same example in Spring. Here, you could do something like what's shown in below. You start with the Car class, but in this case you add an annotation to it, @Autowired:

```java
public class Car {
  @Autowired
  private Engine engine;
  // ...
}
```
<br>

Using the @Autowired annotation tells Spring to automatically inject an object into the reference.

Now let's see the @Component annotation - Annotating a class with @Component tells Spring that it is available for fulfilling injections. In this case, the InlineSixCylEngine would be injected because it is available and satisfies the interface requirement of the association. In Spring, this is called an "autowired" injection:

```java
@Component
public class InlineSixCylinderEngine implements Engine{
  //...
}
```
<br>

#### Aspect Oriented Programming (AOP)
In most applications there are concerns that ‘cut’ across different abstraction layers, the typical example is logging. Cross cutting concerns are concerns that cut across multiple application modules. These concerns often cannot be cleanly decomposed from the rest of the system in both the design and implementation, and can result in either scattering (code duplication), tangling (significant dependencies between systems), or both. Aspect oriented programming help us with the separation of cross cutting concerns and decouple them from the core application modules.

So basically, AOP helps us create application wide services like logging, security, and you apply them in a declarative fashion to the classes that need these services.

## Modules and IoC Container
The Spring Framework consists of features organized into about 20 modules. These modules are grouped into Core Container, Data Access/Integration, Web, AOP (Aspect Oriented Programming), Instrumentation, and Test, as shown in the following diagram.

![IoC Container](/assets/img/iocContainer.png)

The core container is the heart of Spring framework and all other modules are built on top of it. The core container manages how beans are created, manages bean dependencies, provides the dependency injection feature (IoC), contains the BeanFactory (an implementation of factory pattern) which creates and manages the life cycle of the various application objects (known as beans) defined in the Spring bean configuration file. The primary functions of the Spring container are:

- Create and manage objects (IoC)
- Inject object’s dependencies (Dependency Injection)

The container gets its instructions on what objects to instantiate, configure, and assemble by reading configuration metadata.

![Metadata](/assets/img/metadata.png)

As the preceding diagram shows, the Spring IoC container consumes a form of configuration metadata; this configuration metadata represents how you as an application developer tell the Spring container to instantiate, configure, and assemble the objects in your application. Configuration metadata is traditionally supplied in a simple and intuitive XML format, or Java annotations.
Spring configuration consists of at least one and typically more than one bean definition that the container must manage. XML-based configuration metadata shows these beans configured as <bean/> elements inside a top-level <beans/> element. These bean definitions correspond to the actual objects that make up your application.

#### What is a Spring Bean?
Spring Bean is simply a Java object created by the Spring container. It is a Java object that is instantiated, assembled, and otherwise managed by a Spring IoC container. These beans are created with the configuration metadata that you supply to the container.

A Spring application is composed of a set of beans that perform functionality specific to your application layers and are managed by the IoC container.

Beans are uniquely identified by an id attribute, any of the values supplied to the (comma, semicolon, or space separated) name attribute of the bean definition, or even as an alias definition. You can refer to a bean anywhere in the application with id or any of the names or aliases specified in the bean definition. It’s not necessary that you always provide an id or name to the bean. If one isn’t provided, Spring will generate a unique bean name for it; however, if you want to refer to it with a name or an id, then you must provide one.

In short, a Spring bean is an object which Spring framework manages at runtime - management of a Spring bean includes:
- creating an object
- providing dependencies
- intercepting object method calls to provide additional framework features
- destroying an object

Spring is responsible for creating bean objects. But first, you need to tell the framework which objects it should create. We do this by prividing bean definitions. Bean definitions tell Spring \
- which classes the framework should use as beans.
- describe the properties of a bean. 

There are three different ways in which you can define a Spring bean:
1. Annotating your class with the stereotype @Component annotation (or its derivatives).
2. Writing a bean factory method annotated with the @Bean annotation in a custom Java configuration class.
3. Declaring a bean definition in an XML configuration file.

In modern projects, you’re going to use only component annotations and bean factory methods. If it comes to the XML configuration, nowadays Spring allows it mainly for the backward compatibility.

## Spring Context

The Spring context is a registry of all available Spring beans. Classes are identified as Spring beans by annotating them with specific Spring annotations. Examples include @Service, which identifies a business service, @Controller, which identifies a Spring MVC controller (i.e., handles web requests), and @Entity, which is a JPA annotation used to identify classes that are mapped to database tables.

Once these beans are annotated they need to be registered with the Spring context, which Spring Boot does by performing a package scan of all classes in packages in your project.

As the Spring context is being built, it implements the inversion-of-control (IoC) design pattern through dependency injection: when a Spring bean needs a dependency, such as a service or repository, the bean can either define a constructor that accepts the dependent bean or it can leverage the @Autowired annotation to tell Spring that it needs that dependency. Spring resolves all dependencies and "autowires" the application together.
