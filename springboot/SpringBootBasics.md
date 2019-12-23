---
layout: springboot
permalink: /springboot/Basics/
---
## Maven

Maven is a software project management and comprehension tool. With the help of Maven, Spring Boot will be able to configure and install dependencies, compile our Java code and run our class files. Below is how you can install it using Homebrew and check the version to verify for a successful installation.

```shell
brew install maven
mvn --version
```
<br>

## Creating a Spring Boot Application in Spring Tool Suite

Click New -> Spring Starter Project and fill the below information:
- Name: This will be your project name in lowercase 
- Group: This will be a reverse domain name, example com.johndoe.learn
- Artifact: project_name 
- Description: Short project description 
- Package: Same as the group field 

Click Next and select 'Spring Web' under the Available column Click Finish.

Our application will reside in `/src/main/java` , static files and templates will reside in `src/main/resources`.

## Hello World
Open the .java file under src/main/java and add the below code:

```java
@SpringBootApplication
// Add below annotation
@RestController
public class HelloWorldController{
	public static void main(String[] args){
		SpringApplication.run(HelloWorldApplication.class, args);
	}
	
	// Request mapping for root route
	@RequestMapping(value="/", method=RequestMethod.GET)
	public String hello(){
		return "Hello World!";
	}
}
```

Make sure to import the dependencies `CMD + SHIFT + O` in Mac for the @RestController and @RequestMapping annotations to work. Once you run the application, visit localhost:8080 in the browser.

- `@RestController` annotation signifies that the class in our Java file is a Controller in the MVC framework, and allows our controller to respond with data.
- `@RequestMapping` annotation is for mapping web requests onto specific handler classes (class level) and/or handler methods (method level). What we have used above is a method level handler. In the above example, our handler method will respond to the root (“/”) route.

#### Controller
In the Spring framework, a Controller is a class which is responsible for: 
1. Prepare the model/data that needs to be displayed by the view 
2. Choosing the right view to display the data. It can also directly write into response stream by using @ResponseBody annotation and complete the request.

@Controller annotation marks a class as a Spring MVC controller, allowing for the class to be auto-detected through classpath scanning. It is typically used in combination with annotated handler methods based on the @RequestMapping annotation. A Spring MVC controller is used typically in UI based applications where response is generally HTML content. The handler method returns the response “view name” which is resolved to a view technology file (e.g. JSP) by view resolver. And then parsed view content is sent back to browser client.

#### @RestController
@RestController is a shortcut for  @Controller + @ResponseBody. It makes development of RESTful Web Services in Spring framework easier. Response from a web application consists of HTML + CSS + JavaScript, whereas a REST API will return data in form of JSON or XML. The job of @Controller is to create a Map of model object and find a view, but @RestController simply returns the object and it's directly written into HTTP response as JSON or XML, and parsed by client to further process it either for modifying the existing view or for any other purpose.

#### Method vs. Class Level Handler
- **Method Level Handler**: The above HelloWorld example is using a method level handler, meaning that hello() method will handle any request that comes to the “/” path.
- **Class Level Handler**: We can also annotate the whole class so that all methods respond to a certain path. In the below example, the “/hello” route will respond with “Hello World!” and the “/hello/world” route will respond with “Class level annotations are cool too!”.

```java
@RequestMapping("/hello")
public class HelloWorldController{
	@RequestMapping(value="",method=RequestMethod.GET)
	public String hello(){
		return "Hello World!";
	}

	@RequestMapping(value="/world",method=RequestMethod.GET)
	public String world(){
		return "Class level annotations are cool too!";
	}
}
```
<br>
## Query Parameters
To implement query parameters, we need to do use the `@RequestParam` annotation in our methods. We could include the parameter “searchQuery” as a type String (request parameters must always be of String type) and annotate it as a URL query field of “name”. As an example, if we visit localhost:8080?name=John, we can retrieve the value of name as shown below.

```java
@RequestMapping("/")
public String query(@RequestParam(value="name") String searchQuery ){
	return "Hello " + searchQuery;
}
```
<br>
However, if we access our application without the query parameter now, it will crash. We can fix this by including the required=false parameter in the @RequestParam annotation, by default it’s set to true:

```java
@RequestMapping("/")
public String query(@RequestParam(value="name", required=false) String searchQuery){
	return "Hello " + searchQuery;
}
```
<br>

## URL Parameters / Path Variables
Consider this route: localhost:8080/m/59 - Each path variable plays a role to find the exact resource that needs to be displayed to the client. We can use the @PathVariable annotation as shown below:

```java
@RequestMapping("/m/{track}")
public String showLesson(@PathVariable("track") String track){
	return "Track: " + track;
}
```
<br>

## Thymeleaf
Thymeleaf is a templating engine that allows us to write HTML while allowing placeholders for data that will come from Java objects. These placeholders will leverage the **Spring Expression Language** to access the fields of the Java objects from it's getters and setters. Below are the steps we need to follow to use Thymeleaf:

- **Step 1**: Include the Thymeleaf dependency:

```xml
<dependency>
    <groupId>org.thymeleaf</groupId>
    <artifactId>thymeleaf</artifactId>
    <version>3.0.11.RELEASE</version>
</dependency>
```

- **Step 2**: Create your templates (with .html extension) under `src/main/resources/templates`. Thymeleaf expects HTML to be XML compliant, which means all opening tags must have a closing tag or must be self closing.
- **Step 3**: In the controller, return the HTML file name without the filename extension.

#### Thymeleaf Attributes

**th:text**: th:text tells Thymeleaf to replace the content of this element with the value in quotes. Below is the syntax, variable_name and it's value will be coming from the Controller's ModelMap.

```html
<p th:text="${variable_name}"></p>
```
<br>

**th:each**: Let's say in our Controller, we have stored a list of Contacts - each contact has firstName and lastName attributes, and the "contacts" list is stored in ModelMap. In our Thymeleaf template, if we want to loop over "contacts"  and display it as an unordered list, below is how we will do it:

```html
<ul>
	<li th:each="contact: ${contacts}" th:text = "${contact.firstName}"></li>
</ul>
```
<br>

**th:if**: What if we want a certain HTML element/block to only be rendered under certain conditions? We can do so using conditional logic as shown below, where the <p> tag will show up if the contact attribute is empty:

```html
<p th:if="${contacts.size()==0}">No contacts found.</p>
```
<br>

#### Adding Static Assets

1. Create a home for static assets (css, images, javascript, fonts, videos etc). A standard location is `/src/main/resources/static` which is where Thymeleaf will expect to find them while processing our templates.
2. Let's say we want to add a CSS file, style.css - in order to add this in our HTML template, we need to create a self closing `<link>` tag. In the href attribute, instead of adding an absolute path to the CSS file, we can have Thymeleaf discover it by using a link URL expression. To do this, we will prefix the href attribute with `th`. In XML, this th is called a namespace - and may show as an error (name is not bound) by the IDE. To fix this, we have defined the namespace in the html tag as shown below. In the href attribute, the @ sign indicates that a URL should be generated here.

```html
<html lang="en" xmlns:th="http://www.thymeleaf.org">
...

<link rel="stylesheet" th:href="@{/style.css}"   />
<img th:src="@{/img/logo.png}" alt="Test image" />
<script type="text/javascript" th:src="@{/js/lib.js}"></script>
```
<br>

## Sending Information to the view
#### Model
In order to send information from the Controller to the Thymeleaf view, we make use of the `Model`object which exposes a key-value pair in our view. To do this, we inject a Model object in our Controller method:

```java
@RequestMapping("/")
public String home(Model model){
	model.addAttribute("message", "Hello World");
	return 'index';
}
```
<br>

Now, in our view, index.html, we can retrieve the value as shown below:

```html
<p th:text="${message}"></p>
```
<br>

#### ModelMap
ModelMap is also used to pass values to render a view. The advantage of ModelMap is it gives us the ability to pass a collection of values and treat these values as if they were within a Map. 

```java
@RequestMapping("/")
public String hello(ModelMap model){
    model.addAttribute("message", "Hello World");
    mode.addAttribute("name","John");
    return "home";
}
```
<br>

In the above example, we have added two attributes to ModelMap and they will be used in our view as shown below:

```html
<p th:text="${message}"></p>
<p th:text="${name}"></p>
```
<br>

#### ModelAndView
ModelAndView is a holder for both Model and View in the web MVC framework. These two classes are distinct; ModelAndView merely holds both to make it possible for a controller to return both model and view in a single return value. Let's see this in work in the below example:

```java
@RestController
public class MAVExample{
    @GetMapping("/")
    public ModelAndView showPage(){
        ModelAndView modelAndView = new ModelAndView("welcome");
        modelAndView.addObject('message','Hello World');
    }
    return modelAndView;
}
```
<br>

Below is our view:

```html
<!-- welcome.html -->
<p th:text="${message}"></p>
```
<br>

## Sessions
To use session, we need to include the `HttpSession` object. In the below example, we are creating a session variable named “count” and setting it’s value to 0 by using the `setAttribute()` method on the session object. Similarly, we can use the `getAttribute()` method to retirve the value and cast it to the appropriate type.

```java
public String index(HttpSession session){
        session.setAttribute("count", 0);
        // Retirving the value of session object
        Integer count = (Integer) session.getAttribute("count");
    }
```
<br>

In order to pass the session value to a view, use the addAttribute() method of the Model object, as explained in the previous section.

You can invalidate a session using the `session_variable.invalidate()` method.

## Handling Form Data
- **Step 1**: In the form's action attribute, list the RequestMapping route that will process form data.
- **Step 2**: In the @RequestMapping annotation, add `method=RequestMethod.POST`
- **Step 3**: Use the `@RequestParam(value="username") String username,..` annotation in our method to retrieve form data.
- **Step 4**: Save any relevant form data in the session and redirect using `return: "redirect:/routename"`

## Flash Data
Flash data is data that only persists across the next request. This sort of session data is very useful for things such as error messages, success notification, or anything else that you would want to show a user only immediately following their request.

- **Step 1**: Add `RedirectAttributes` in our method
- **Step 2**: Use `addFlashAttribute(key, value)` method on the redirectAttribute - here, key is the name of the error and value is the message we want to be flashed.
- **Step 3**: In the redirected page, you can display the message using Thymeleaf attributes.

```java
@RequestMapping("/createError")
public String flashMessage(RedirectAttributes, redirectAttributes){
	redirectAttributes.addFlashAttribute("key","value");
	return "redirect:/";
}
```
<br>





