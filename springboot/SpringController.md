---
layout: springboot
permalink: /springboot/SpringController/
---
## Spring Controller

In any Spring web application, a Controller is a Java object who's job is to handle requests to certain URIs. We will use annotations to methods in this class to indicate which URI each method should handle. 

1. Let's first create a controller package within com.teamtreehouse.giflib
2. Within the controller package, create the Java class for the controller and give is a name, say GifController.
3. To indicate that this class is a Spring Controller, we are using the `@Controller` annotation on the class itself.
4. In a Spring controller, we can add as many methods as we like. We've added a method to handle request to our applications home page.
5. To execute the method when anyone visits the home page, we've added the `@RequestMapping()` annotation, which maps a URI to a controller method.
6. The  `@ResponseBody` annotation indicates that the String we are returning from the method should be used as the response without any further processing.
7. In order for Spring to automatically scan packages for controllers, we need to add the `@ComponentScan` annotation to our AppConfig class.

```java
package com.teamtreehouse.giflib.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class GifController {

    @RequestMapping(value = "/")
    @ResponseBody
    public String listGifs(){
        return "List of all the GIF's";
    }
}
```
<br>

## Thymeleaf

Thymeleaf is a templating engine that allows us to write HTML while allowing placeholders for data that will come from Java objects. These placeholders will leverage the **Spring Expression Language** to access the fields of the Java objects from it's getters and setters.

1. Add the dependency in build.grade: `org.springframework.boot:spring-boot-starter-thymeleaf:2.2.2.RELEASE`
2. Create your templates (with .html extension) under `src/main/resources/templates`
3. Thymeleaf expects HTML to be XML compliant, which means all opening tags must have a closing tag or must be self closing.
4. In our controller method remove the @ResponseBody annotation, as we do need further processing by Thymeleaf before the response is sent to the client.
5. In the controller, return the HTML file name without the filename extension.

That's all we need to serve HTML templates!. Below is the high level workflow of how our application is working:

- The server's dispatcher servlet receives the request from the client.
- The dispatcher servlet examines the URI, and determines which controller and which of it's methods is mapped to the URI by using the @RequestMapping annotation.
- The mapped controller method is executed and puts together a response to the original HTTP request.
- This response is now fed to the view resolver. Since our application is configured to use Thymeleaf, all the data is passed on to the Thymeleaf template and final response is handed over the dispatcher servlet.
- The dispatcher servlet hands it back to the web server. The web server creates an HTTP response with a status code of 200 and includes the HTML from our application in the response body.
- The entire response is then sent in response to the original HTTP request.

## Adding Static Assets

1. Create a home for static assets (css, images, javascript, fonts, videos etc). A standard location is `/src/main/resources/static` which is where Thymeleaf will expect to find them while processing our templates.
2. Let's say we want to add a CSS file, style.css - in order to add this in our HTML template, we need to create a self closing `<link>` tag. In the href attribute, instead of adding an absolute path to the CSS file, we can have Thymeleaf discover it by using a link URL expression. To do this, we will prefix the href attribute with `th`. In XML, this th is called a namespace - and may show as an error (name is not bound) by the IDE. To fix this, we have defined the namespace in the html tag as shown below. In the href attribute, the @ sign indicates that a URL should be generated here.

```html
<html lang="en" xmlns:th="http://www.thymeleaf.org">
...

<link rel="stylesheet" th:href="@{/style.css}"   />
```
<br>

