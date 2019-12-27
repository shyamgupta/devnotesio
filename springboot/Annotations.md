---
layout: springboot
permalink: /springboot/Annotations/
---
In this section we will learn about some useful annotations associated with the Spring framework.

## Stereotype Annnotations: @Component
During initial release of Spring, all beans  used to be declared in an XML file. For a large project, this quickly becomes a massive task. In later versions of Spring, annotation-based dependency injection and Java-based configuration were provided which automatically scans and register classes as Spring bean that are annotated with @Component. Spring is able to auto scan, detect and instantiate your beans from pre-defined project package, no more tedious beans declaration in in XML file. Applying this annotation to class informs Spring that the class is a component and an object of this class can be instantiated and injected into another component.

***@Component is a generic stereotype for any Spring-managed component. @Repository, @Service, and @Controller are specializations of @Component*** for more specific use cases, for example:
- **@Service**: Denotes that the class provides some services. A service is any class that provides methods to interact with domain logic or external components without maintaining state that changes the overall behavior of the service. For example, a service may act on behalf of an application to obtain documents from a database or obtain data from an external REST API.
- **@Repository**: This annotation indicates that the class deals with CRUD operations
- **@Controller**: Mostly used with web applications or REST web services to specify that the class is a front controller and responsible to handle user request and return appropriate response.

Spring @Component annotation is used to denote a class as Component. It means that Spring framework will autodetect these classes for dependency injection when annotation-based configuration and classpath scanning is used. 

By using the above annotations, Spring creates automatic wiring in these two ways:
- **Component scanning**: In this, Spring automatically searches the beans to be created in the Spring IoC container
- **Autowiring**: In this, Spring automatically searches the bean dependencies in the Spring IoC container

Spring needs to know which packages to scan for annotated components in order to add them to the IoC container. In a Spring Boot project, we typically set the main application class with the @SpringBootApplication annotation. Under the hood, @SpringBootApplication is a composition of the @Configuration,  @ComponentScan, and @EnableAutoConfiguration annotations. With this default setting, Spring Boot will auto scan for components in the current package (containing the @SpringBoot main class) and its sub packages.

Let's see the following TransferService interface:

```java
public interface TransferService { 
      void transferAmmount(Long a, Long b, Amount amount); 
} 
```
<br>

Its implementation  TransferServiceImpl is annotated with the @Component annotation. This annotation is used to identify this class as a component class, which means, it is eligible to scan and create a bean of this class. Now there is no need to configure this class explicitly as a bean either by using XML or Java configuration--Spring is now responsible for creating the bean of the TransferServiceImpl class, because it is annotated with @Component:

```java
@Component 
public class TransferServiceImpl implements TransferService { 
	@Override 
	public void transferAmmount(Long a, Long b, Amount amount) { 
		//business code here 
	} 
} 
```
<br>

In Spring, you have to enable component scanning in your application, because it is not enabled by default. You have to create a configuration Java file, and annotate it with @Configuration and @ComponentScan. This class is used to search out classes annotated with @Component, and to create beans from them. Since these annotations are so frequently used together (especially if you follow the best practices above), Spring Boot provides a convenient @SpringBootApplication alternative.
The @SpringBootApplication annotation is equivalent to using @Configuration, @EnableAutoConfiguration and @ComponentScan with their default attributes: [...]

The @SpringBootApplication annotation includes the following annotations:
- **@Configuration** informs Spring that our HelloWorldController class contains configuration information. (This annotation can be used to create beans that will get registered with the Spring context.) 
- **@EnableAutoConfiguration** tells Spring to automatically configure resources from dependencies found in the CLASSPATH, such as H2 and Tomcat.
- **@ComponentScan** tells Spring to scan packages in the CLASSPATH under the current package for Spring-annotated components such as @Service and @Controller.

Spring scans the CLASSPATH and automatically creates components. It then populates the Spring context with the application components found in the package scan.

## @Autowired
Spring provides support for automatic bean wiring. This means that Spring automatically resolves the dependencies that are required by the dependent bean by finding other collaborating beans in the application context. Bean Autowiring is another way of DI pattern configuration. 

Spring's @Autowired annotation is used for auto bean wiring. This @Autowired annotation indicates that autowiring should be performed for this bean.

In our previous example, we have TransferService - let's say it has dependencies of AccountRepository and TransferRepository. Its constructor is annotated with @Autowired indicating that when Spring creates the TransferService bean, it should instantiate that bean by using its annotated constructor, and pass in two other beans, AccountRepository and TransferRepository, which are dependencies of the TransferService bean. Let's see the following code:

```java
@Component
public class TransferServiceImpl implements TransferService{
	AccountRepository accountRepository; 
    TransferRepository transferRepository; 

    @Autowired 
    public TransferServiceImpl(AccountRepository accountRepository, TransferRepository transferRepository) { 
      super(); 
      this.accountRepository = accountRepository; 
      this.transferRepository = transferRepository; 
    } 
}
```
<br>

@Autowired annotation handles only wiring part - we still have to define the beans (as shown above) so the container is aware of them and can inject them for us

Note--As of Spring 4.3, the @Autowired annotation is no more required if you define only one construct with arguments in that class. If class has multiple argument constructors, then you have to use the @Autowired annotation on any one of them.

The @Autowired annotation is not limited to the construction (as shown above); it can be used with the setter method, and can also be used directly in the field, that is, an autowired class property directly.