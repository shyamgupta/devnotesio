---
layout: springboot
permalink: /springboot/ModelingData/
---

## Modeling Data with POJOs

In this example, we are developing a web application that will display GIF's. Our GIF's will be represented as POJO's with the following attributes:

- Name (String), 
- Date uploaded (Date object),
- Username who uploade the GIF(String)
- Whether or not the GIF is marked as a favorite (Boolean)

The Java class we will use to represent each GIF object is referred to as a model. To organize all potential models, we can create a model package within the root package of our project. Within the model package, we have created the below model Java class:

```java
//Gif.class
package com.teamtreehouse.giflib.model;

import java.time.LocalDate;

public class Gif {

    //fields
    private String name;
    private LocalDate dateUploaded;
    private String username;
    private boolean favorite;

    //Constructors

    public Gif(String name, LocalDate dateUploaded, String username, boolean favorite) {
        this.name = name;
        this.dateUploaded = dateUploaded;
        this.username = username;
        this.favorite = favorite;
    }

    //Getters and Setters
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public LocalDate getDateUploaded() {
        return dateUploaded;
    }

    public void setDateUploaded(LocalDate dateUploaded) {
        this.dateUploaded = dateUploaded;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public boolean isFavorite() {
        return favorite;
    }

    public void setFavorite(boolean favorite) {
        this.favorite = favorite;
    }
}
```
<br>

With our POJO model in place, we want to accomplish the following:
1. We want to store our GIF data (as static GIF files) in our GIF objects.
2. Feed the GIF objects to our Thymeleaf template.
3. Use the Spring Expression Language in the Thymeleaf templates to access the objects.

To do this, we have added the gifDetails() method in our Controller:

```java
@RequestMapping("/gif")
public String gifDetails(ModelMap modelMap){

	//Create a GIF object that we can make available to the view
	//gif-name is the name of the GIF file in the static folder
	Gif gif = new Gif("gif-name", LocalDate.of(2019,12,25),"Chris",true);

	//Make our newly created object available to our gif-details view
	modelMap.put("gif", gif);

	// we are returning the Thymeleaf template
	return 'gif-details';
}
```
<br>

To make our newly created GIF object available to the Thymeleaf template, we are making use of the `ModelMap` interface - it is used to pass values to render a view. The advantage of ModelMap is it gives us the ability to pass a collection of values and treat these values as if they were within a Map. There's no need to add ModelMap in the return value, as the Spring framework will take care of making it available to Thymeleaf.

Now, let's turn our attention to the view:

```html
<!-- gif-details.html in templates folder -->

<img th:src="@{'/gifs/' +${gif.name} +'.gif'}" alt="">

```
<br>

Note how we are accessing our object (available from the ModelMap) by using `${object_name.field_name}`. Here, Thymeleaf is "assuming" the name of the getter -  for our "name" field, the getter name is getName() which uses the default naming convention for getters and setters.

## Repository

So far our application displays only one GIF which is hard coded in the controller- lets update it to display a GIF we can select from the repository. A repository is a collection we have stored in memory or database.

1. Create a repository package within the root package of your project.
2. Within the repository package, create the GifRepository Java class. This class will act as both storage of GIF objects and methods for interacting with those objects.
3. We will store our GIF objects in a static Java list. In this example we are pre-defining our GIF's (in a production app, they will come from a database). We are using the `asList()` method from the `Arrays` class to which we can add any number of GIF objects.
4. Since the list is private, we have included Getters to expose data

```java
package import com.teamtreehouse.giflib.repository

@Component
public class GifRepository{
	private static final List<Gif> ALL_GIFS = Arrays.asList(
			new Gif("android-explosion", LocalDate.of(2015,2,13), "Chris Ramacciotti", false),
			new Gif("ben-and-mike", LocalDate.of(2015,10,30), "Ben Jakuben", true),
			new Gif("book-dominos", LocalDate.of(2015,9,15), "Craig Dennis", false),
			new Gif("compiler-bot", LocalDate.of(2015,2,13), "Ada Lovelace", true),
			new Gif("cowboy-coder", LocalDate.of(2015,2,13), "Grace Hopper", false),
			new Gif("infinite-andrew", LocalDate.of(2015,8,23), "Marissa Mayer", true)
	);
}

//Getter
public Gif findByName(String name){
	for (Gif gif: ALL_GIFS){
		if(gif.getName().equals(name)){
			return gif;
		}
	}
	return null; //if GIF is not found
}
```
<br>

Now let's update our controller to use the above repository - in order to do that, we will need a reference to the GifRepository object. 

1. Since we will likely access it from any controller method, we are adding it as an instance (private) field in our Controller class.
2. Without assigning anything to gifRepository, if we try to call any of it's methods, we will encounter a Null Pointer Exception. Spring can initialize fields for us as long as it can find a Spring component of the same class as that of the field. Since we need a Spring component of GifRepository - to tell Spring that we want to auto assign our field, we use the `@Autowired` annotation. In order to tell Spring that GifRepository is a valid Spring component, we have added the `@Component` annotation to the GifRepository class. 


```java
public class GifController{

	@Autowired
	private GifRepository gifRepository;


	@RequestMapping("/gif")
	public String gifDetails(ModelMap modelMap){
		Gif gif = gifRepository.findByName("android-explosion");
		modelMap.put(gif);
		return 'gif-details';
	}

}
```
<br>

## @PathVariable

We will now update our controller method serve the GIF object that corresponds to the name included in the URI. We can do so using the `@PathVariable` annotation in our method as shown below:

```java
@RequestMapping("/gif/{name}")
	public String gifDetails(ModelMap modelMap, @PathVariable String name){
		Gif gif = gifRepository.findByName(name);
		modelMap.put(gif);
		return 'gif-details';
	}
```





