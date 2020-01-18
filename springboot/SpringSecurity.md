---
layout: springboot
permalink: /springboot/SpringSecurity/
---

## Basic Auth
Basic Auth is the simplest protocol available for performing web service authentication over HTTP protocol. 
- Basic Auth requires a username and password. 
- The client calling the web service takes these two credentials, converts them to a single Base 64–encoded value and passes it along in the Authentication HTTP header.
- The server compares the credentials passed to those stored. If it matches, the server fulfills the request and provides access to the data. 
- If the Authentication HTTP header is missing or the password doesn’t match the user name, the server denies access and returns a 401 status code, which means the request is Unauthorized.

## Spring Security
Spring Security is a part of the Spring Framework and provides authentication, authorization and other security features for Spring-based applications.
- `spring-boot-starter-security` - Maven dependency that adds security module.
- `@EnableWebSecurity` - Annotation that enables Spring Security’s support.

In the below steps, we will see how to use Spring Security and Basic Auth to secure our APIs.

**Step 1: Maven Dependencies**

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-security</artifactId>
</dependency>
```
<br>

**Step 2: Create a Java Class that extends WebSecurityConfigurerAdapter**

In the root package, create a config folder - within this, create a Java class that extends WebSecurityConfigurerAdapter
- We've used the `@Configuration` annotation to indicate that this is a configuration class.
- We've used `EnableWebSecurity` annotation to enable Spring Security support
- We are overriding WebSecurityConfigurerAdapter's configure() method, where we are specifying that we want to authorize all requests to our API and that we want to use Basic Auth.
- In the configureGlobal() method, we are creating an in-memory user and specifying it's password.
- Now, when you run the application and visit the endpoint in a browser, you'll be prompted for the username & password. If you're using Postman, specify the type as "Basic Auth" and provide credentials in the Authorization tab.


```java
package com.shyamgupta.config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;

@Configuration
@EnableWebSecurity
public class SpringSecurityConfig extends WebSecurityConfigurerAdapter{
	@Override
	protected void configure(HttpSecurity http) throws Exception{
		http
		.csrf().disable()
		.authorizeRequests()
		.anyRequest()
		.authenticated()
		.and()
		.httpBasic();
		
	}
	
	@Autowired
	//method creates an in-memory user and password
	public void configureGlobal(AuthenticationManagerBuilder auth)
		throws Exception{
		auth.inMemoryAuthentication()
		.withUser("admin")
		.password(encoder().encode("password"))
		.roles("USER");
		
	}
	
	@Bean
	public PasswordEncoder encoder() {
		return new BCryptPasswordEncoder();
	}
}
``` 