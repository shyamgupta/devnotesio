---
layout: springboot
permalink: /springboot/ConsumingREST/
---
## RestTemplate

When we visit the below URL, we get a JSON response:

https://official-joke-api.appspot.com/random_joke 

```json
{
	"id":283,
	"Type":"general",
	"setup": "What's the best thing about elevator jokes?",
	"punchline":"They work on so many Levels."
}
```
<br>

This format is not very useful when fetched through a browser or through cURL. A more useful way of consuming REST services is programmatically - to do this, Spring provides `RestTemplate`. We will build an application that consumes the the above publically accessible jokes API. Basically, RestTemplate is used to make HTTP Rest Calls (REST Client).If we want to make an HTTP Call, we need to create an HttpClient, pass request and form parameters, setup accept headers and perform unmarshalling of response, all by yourself, Spring Rest Templates tries to take the pain away by abstracting all these details from you

RestTemplate provides an easy way to access a REST service. All it needs is:
- URL template: The URL template string may contain parameters in the same way as the request mapping in the Spring controllers, the name of the parameter being between the { and } characters
- a type that is used to convert the result
- and a Map object with the parameters

You can use RestTemplate in the following ways:

- Create an instance at the point you need it:

```java
RestTemplate rest = new RestTemplate();
```
<br>

- Declare it as a bean in the main class (which contains the public static void main(String[] args) function) (and inject it where you need it. There are several ways to configure this bean and we are going to use the default `RestTemplateBuilder`:

```java
@Bean
public RestTemplate restTemplate(RestTemplateBuilder builder) {
      return builder.build();
  }
```
<br>

The RestTemplate class also provides aliases for all supported HTTP request methods, such as GET, POST, PUT, DELETE, and OPTIONS. Below are some of the useful methods associated with RestTemplate - Methods contains "Method Name" and "what this method will return". For example, getForObject() performs a GET call, converts the HTTP response into an object and returns that object:

<table class="table table-striped table-bordered table-hover table-responsive-sm">
	<thead class="bg-danger text-light">
		<tr>
			<th>HTTP Verb</th>
			<th>RestTemplate Method</th>
			<th>Description</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>GET</td>
			<td><code>getForObject(url, classType)</code></td>
			<td>URL is the 3rd party REST API URL, and the classType is the Java class we will use to represent the JSON response</td>
		</tr>
		<tr>
			<td>POST</td>
			<td><code>postForObject(url, request, classType)</code></td>
			<td>POSTs the given object to the URL, and returns the representation found in the response as given class type.</td>
		</tr>
		<tr>
			<td>PUT</td>
			<td><code>put(url,request)</code></td>
			<td>PUTs the given request object to URL</td>
		</tr>
		<tr>
			<td>DELETE</td>
			<td><code>delete(url)</code></td>
			<td>Deletes the resource at the specified URL.</td>
		</tr>
	</tbody>
	
</table>

**Step 1 - Maven Dependencies**

Create a new Spring Boot application with the following dependencies:

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
	<groupId>com.jwebmp.jpms.jackson.core</groupId>
	<artifactId>jackson-databind</artifactId>
	<version>0.68.0.1</version>
</dependency>
```
<br>

**Step 2 - Create a Java Class that represents the response from the 3rd party API**

- This class will represent the response received from the 3rd party API. ***To directly bind your data to your custom types, you need to specify the variable name to be exactly the same as the key in the JSON document returned from the API.***.
- The JSON response then needs to be converted to Java. To bind the JSON data to the Jokes type, we are using Jackson - it serializes (or maps) Java objects JSON and vice versa. We've added the Jackson dependency in our pom.xml.
- In this class, we are also overriding the `toString()` method which represents an object of a class as a String, i.e. returns a textual representation of the object. When you print an object, by default the Java compiler invokes the toString() method on the object, and will return an output similar to `com.example.toString.Joke.@6d06d69c`. Looking at this output, we can see that it doesn't give us much information about the contents of our Joke object. Generally, we aren't interested in knowing the hashcode of an object, but rather the contents of our object's attributes. By overriding the default behavior of the toString() method, we can make the output of the method call more meaningful. Using the toString() method also helps when you want to print the current state of an instance, using a logging framework.

```java
package com.example.entity;
public class Joke {
	private Long id;
	private String type;
	private String setup;
	private String punchline;
	
	//constructors
	public Joke() {}

	public Joke(Long id, String type, String setup, String punchline) {
		super();
		this.id = id;
		this.type = type;
		this.setup = setup;
		this.punchline = punchline;
	}

	//getters and setters
	public Long getId() {
		return id;
	}

	public void setId(Long id) {
		this.id = id;
	}

	public String getType() {
		return type;
	}

	public void setType(String type) {
		this.type = type;
	}

	public String getSetup() {
		return setup;
	}

	public void setSetup(String setup) {
		this.setup = setup;
	}

	public String getPunchline() {
		return punchline;
	}

	public void setPunchline(String punchline) {
		this.punchline = punchline;
	}

	@Override
	    public String toString() {
	        return  "Joke{" + "id=" + id + ", type=" + type+ ", setup=" + setup + ", punchline=" + punchline + "}";
	    }

}
```
<br>

**Step 3 Use RestTemplate in your main application**
In our main application, we have the following:
- A logger, to send output to the log (the console, in this example).
- A RestTemplate, which uses the Jackson JSON processing library to process the incoming data.
- A CommandLineRunner that runs the RestTemplate (and, consequently, fetches our quotation) on startup.

```java
package com.example;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.client.RestTemplateBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;
import com.example.entity.Joke;


@SpringBootApplication
public class ConsumingApplication {
	
	
	private static final Logger log =  LoggerFactory.getLogger(ConsumingApplication.class);

	public static void main(String[] args) {
		SpringApplication.run(ConsumingApplication.class, args);
	}
	
	@Bean
	public RestTemplate restTemplate(RestTemplateBuilder builder) {
		return builder.build();
	}
	
	@Bean
	public CommandLineRunner run(RestTemplate restTemplate) throws Exception{
		return args -> {
			Joke joke = restTemplate.getForObject(
					"https://official-joke-api.appspot.com/random_joke",Joke.class);
			log.info(joke.toString());
		};
	}

}
```

