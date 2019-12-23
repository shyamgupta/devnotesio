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

#### Inversion of Control (IoC)
When developing an application, you always have dependencies between and on components, services, classes etc. Without Inversion of Control you would ‘wire’ these together on the spot where you would need the dependency. The disadvantage of this is that when you would like to use a different implementation of your dependency, you are forced to change your code. With Spring, wiring of these dependencies is taken out of the code, and an external party manages the wiring, namely the container. Hence the name inversion of control, you let something from the outside control how your dependencies are wired together. We speak of dependency injection because the container ‘injects’ the necessary dependencies instead of letting the developer manage them.

Spring provides an object factory - our application can talk to Spring requesting for a new object. Based on the configuration file or annotation, Spring will give us the appropriate implementation. This makes our application configurable.

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

Bean definition contains the information called configuration metadata, which is needed for the container to know the following:

- How to create a bean
- Bean’s lifecycle details
- Bean’s dependencies

A Spring application is composed of a set of beans that perform functionality specific to your application layers and are managed by the IoC container.

Beans are uniquely identified by an id attribute, any of the values supplied to the (comma, semicolon, or space separated) name attribute of the bean definition, or even as an alias definition. You can refer to a bean anywhere in the application with id or any of the names or aliases specified in the bean definition. It’s not necessary that you always provide an id or name to the bean. If one isn’t provided, Spring will generate a unique bean name for it; however, if you want to refer to it with a name or an id, then you must provide one.
