---
layout: springboot
permalink: /springboot/Setup/
---
## Classpath and JAR Files

The concept of a path should be familiar to anyone who has worked on a DOS or Unix platform. It’s an environment variable that provides an application with a list of places to look for some resource. The most common example is a path for executable programs. The Java CLASSPATH environment variable, similarly, is a list of locations that can be searched for packages containing Java class files. Both the Java interpreter and the Java compiler use CLASSPATH when searching for packages and classes on the local host. A location on the class path can be a directory name or the name of a class archive file.

A JAR (**J**ava **Ar**chive) file is a bundle of compiled Java code.  Java supports archives of class files in its own Java archive ( JAR) format, and in the conventional ZIP format. JAR and ZIP are really the same format, but JAR archives include extra files that describe each archive’s contents. JAR files are created with the SDK’s jar utility. The archive format enables large groups of classes to be distributed in a single file; the Java interpreter automatically extracts individual class files from an archive, as needed. Once the JAR file is downloaded, all we need to do is to add it to the classpath - we can then start interacting with the classes and objects defined in the JAR.

## Gradle

Gradle is a general purpose build management system that supports automatic download and configuration of dependencies or other libraries (in addition to compiling and executing your code, testing and building your project for deployment). A project using Gradle describes its build via a `build.gradle` file. This file is located in the root folder of the project and we will use it to list the libraries we want to incorporate into our project. After this is done, with the click of a button in our IDE, Gradle will download all the JAR files for the libraries we listed, including their dependencies.

Below is a sample `build.gradle` file

```
repositories {
	mavenCentral()
}
```
<br>
- **repositories**: The repositories block allows us to define one or more locations where Gradle should look for the libraries we list. We will be using Maven Central which is one of the largest public dependency repositories.
- **dependencies**: This is the block where we will be listing the libraries we need. Whenever we update this block, we can refresh the projects jar files through the Gradle tool window available in most IDEs. Below is the syntax to list a library we need - the `compile` keyword tells Gradle that we need the dependency at compile time, i.e. it's required for our source code to compile:

```
compile 'groupId:artifactId:version'
//Example
compile 'org.springframework:spring-webmvc:5.2.2.RELEASE'
```

<br>
Gradle simplifies dependency management - for example the spring-mvc library has 6 dependencies and each of these have their own dependencies & so on. Installing these dependencies manually is not practical, and that's where Gradle comes to the rescue.

## Gradle for Spring Boot Development

- **apply plugins**: In Gradle, plugins are extensions of Gradle and add functionalities to Gradle's built-in features. The java plugin allows Gradle to compile different source sets, compiling main code, compile and run test code separately. The java plugin expects our main application code in the `/src/main/java` directory.
- **dependencies**: Spring Boot allows us to create a standalone, runnable Spring application that has an embedded Tomcat web server. The Spring Boot dependency we've added includes the spring-mvc dependency.
- **plugins**: Spring Boot has a Gradle plugin that can deploy a Spring application with a web server. However, this plugin is not built into Gradle and we need to add it to the buildscript. Additionally, the `org.springframework.boot` entry under apply plugin will allow us to run a Gradle task that will compile and deploy our application in an embedded web server.


```
plugins {
    id 'java'
    id 'org.springframework.boot' version '2.2.2.RELEASE'
}

group 'org.teamtreehouse'
version '1.0-SNAPSHOT'


apply plugin: 'java'
apply plugin: 'org.springframework.boot'

sourceCompatibility = 1.8

repositories {
    mavenCentral()
}

dependencies {
    testCompile group: 'junit', name: 'junit', version: '4.12'
    compile 'org.springframework.boot:spring-boot-starter-web:2.2.2.RELEASE'

}
```
<br>
## Configure Your Application

1. Create a base package under which we will put all of our Java code. Under `/src/main/java` create a new package and call it `com.teamtreehouse.giflib` for example.
2. Within the package we created above, create Java class named `AppConfig` (it could be any name).
3. Below is the code in our AppConfig file - it has the main() method. Within the main() method, we are calling the `SpringApplication.run()` method in the Spring Boot library that will run our application, thereby creating a new application context.
4. Finally, in order for Spring to be auto configured, we have added the `@EnableAutoConfiguration` annotation. This annotation tells Spring Boot to “guess” how you want to configure Spring, based on the jar dependencies that you have added. Since spring-boot-starter-web dependency added to classpath leads to configure Tomcat and Spring MVC, the auto-configuration assumes that you are developing a web application and sets up Spring accordingly.
5. In order for Spring to automatically scan packages for controllers, we need to add the `@ComponentScan` annotation to our AppConfig class.

```java
package com.teamtreehouse.giflib;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@EnableAutoConfiguration
@ComponentScan
public class AppConfig {
    public static void main(String args[]){
        SpringApplication.run(AppConfig.class, args);
    }
}
```
<br>
Above steps are all we need to configure a Spring application which is deployable to a web server. When you run the application and visit `http://localhost:8080/`, you should see the Whitelabel Error Page.

