---
layout: springboot
permalink: /springboot/GraphQL/
---
## Introduction
GraphQL is a query language for API's or a syntax that describes how to ask for data and is generally used to load data from a server to a client. It is an alternative to REST and allows us to request data in the exact shape we want, and also make changes to our data. GraphQL is composed of two main parts:
- GraphQL Query Language (which is what we will be focusing on) is used to request data
- Framework which processes the queries.

GraphQL benefits:
- Allows us to specify the format in which we want our data
- Self documenting API, i.e. by looking at the API's schema, what the data coming out of if will look like, which queries will work & which one's wont. The schema looks a lot like JSON.
- Ability to fetch deeply nested data in a single request. 
- Ability to mount GraphQL in front of any existing API.
- Super flexible - for developing new API's, we can start with scratch where our GraphQL server that communicates with our database. However, if we alreday have an API that communicates with the database, you can bolt GraphQL in front of that API.
- GraphQL does not care where the data comes from, making it useful for a wide variety of applications
- GraphQL is language agnostic, as its just a query language.

With REST, the client hits an endpoint and gets a massive JSON object as the response. With GraphQL, instead of hitting a URL endpoint, GraphQL lets you write a query to request exactly the data we want and receive just that in the response.

## REST vs. GraphQL
Let's say we want to retrieve information related to authors: name, courses authored, rating, most recent (3) topics covered. With the REST approach, we will need to create the following endpoints:

- /author/<id> : Provides the author information for a given id. This may also return additional fields we may not need.
- /author/<id>/courses
- /author/<id>/rating
- /author/<id>/topics: Will retrieve all topics instead of 3 most recent topics we are interested in.

Instead of having multiple endpoints that return fixed data structures, GraphQL APIs typically only expose a single endpoint.  This works because the structure of the data that’s returned is not  fixed. Instead, it’s completely flexible and lets the client decide what data is actually needed. That means that the client needs to send more information to the server to express its data needs - this information is called a query.

With GraphQL, we can accomplish the above in a single request, by composing a query - the query (shown below) will ask for exactly what we need and we will receive the JSON object with the just the information requested. Note that we are only requesting the information we are interested in, for example the author name, course title etc - this means that the client does not have to filter the results (as with REST) to get what was requested, and its fast. All of this is done in just one query instead of multiple round trips.

```graphql
query{
    author(id:2000){
        name
        courses{
            title
        }
        rating
        topics(last: 3){
            name
        }
    }
}
```
<br>

<table class="table table-striped table-bordered table-hover table-responsive-sm">
	<thead class="bg-danger text-light">
		<tr>
			<th>REST</th>
			<th>GraphQL</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>Multiple round trips to collect the information from multiple sources.</td>
			<td>One single request to collect the information by using the GraphQL query.</td>
		</tr>
		<tr>
			<td>
				Over fetching & under fetching of data resources. We end up with data we don't need (as there's no way to filter out the request) or if we want something specific which is not part of the endpoint resulting in multiple round trips.
			</td>
			<td>
				You get what you asked for by using tailor made queries.
			</td>
		</tr>
		<tr>
			<td>Front end teams rely heavily on backend teams to deliver APIs - the front end development could be stalled until the API is ready.</td>
			<td>Frontend and backend teams can work independently, where the front end team can work on mock APIs, until the backend API is available.</td>
		</tr>
		<tr>
			<td>Caching is built into HTTP.</td>
			<td>Does not use HTTP for caching and its upto the developer to implement caching.</td>
		</tr>
	</tbody>
</table>

## GraphQL Ecosystem and Tools
- **GraphQL Clients**: 
    * Handles sending queries to the server, and receiving the JSON from the server. 
    * Integrates with our view and updates the UI. 
    * Handles caching the query results.
    * Error handling and schema validation
    * Some clients also provide local state management, caching, pagination
    * Popular GraphQL clients: Apollo (very popular), Relay (only for React)
- **GraphQL Server**:
    * Primary purpose: Receive the query from the client and responds back. 
    * We write Schema and Resolver functions on the GraphQL server. Resolver functions resolve a value for a type/field in the GraphQL schema. Resolvers can return scalars or objects. 
    * GraphQL Execution Engine parses the query from the client, validates the schema and returns the JSON response back. It also executes resolvers for each field.
    * Popular GraphQL server libraries: Apollo Server (very popular), Express GraphQL, GraphQL Yoga
- **Database to GraphQL Server**:  Prisma is a popular database to GraphQL server option, and it sits between the GraphQL server and the actual database. It bridges the gap between the database and resolvers, supports both SQL and NoSQL databases. It replaces the traditional ORM - it provides a data access layer that makes it easy for API servers to talk to database through Prisma.
- **GraphQL Tools**:
    * GraphiQL: Its an in-browser IDE for writing, validating and testing GraphQL queries. Supports live syntax and validation errors. GraphiQL is a simple web app that is able to communicate with any GraphQL Server and execute queries and mutations against it.
    * GraphQL Voyager: Represents a GraphQL API as an interactive graph. Also used during design of data model.
    * GraphQL Faker: Mock your API with realistic data from faker.js
    * GraphQL Visual Editor: Create schemas by joining visual blocks and the Visual Editor will transform them to actual code.

## GraphQL Terminology
- **Queries**: Queries specify the data we want to fetch from a GraphQL server.
- **Fields**: They are properties that make up the shape of our objects. Using fields, we can include or exclude properties of an object to craft a tailored response.
- **Types**: Types are collection of fields that make up a queryable object. 
- **Schema**: The GraphQL schema defines the data points offered via an API. The schema contains the data types and relationships between them and the set of operations available, things like queries for retrieving data and mutations for creating, updating, and deleting data. ***Schema files names must end with `graphqls`. There can only be one root Query and one root Mutation per schema.***
- **Resolvers**: Let's say our API is able to run two query operations, i.e one to retrieve an array of books and another to retrieve a book based on its id. The next step for us is to define how these queries get resolved so that the right fields are returned to the client. The way to do this is by defining a resolver function for every field in the schema.

For example, if I've  movie and studio types, the movie field will have fields like name, release data, and studio type will have fields like an ID, name etc.

## GraphQL Schema
Schema describes the functionality available to the client applications that connect to it. Schema defines the server's API, allowing clients to know which operations can be performed by the server. The schema is written using the GraphQL schema language (also called schema definition language, SDL). With it, you can define object types and fields to represent data that can be retrieved from the API as well as root types (query, mutation) that define the group of operations that the API allows. Generally, a schema is simply a collection of GraphQL types, with special root types (when developing an schema for an API).

In the below example, we have defined a Book type with four fields and a root Query type with two fields. The two fields in the root Query type defines what queries/operations the server can execute. The books field returns a list of Book type, and the book field will return a Book type based on the id passed as an argument to the book query.

```graphql
type Book {
    id: Int!
    title: String!
    pages: Int
    chapters: Int
  }

type Query {
	books: [Book!]
	book(id: Int!): Book
}
```
<br>

## Types
GraphQL has a strongly typed schema - this schema acts as a contract between the client and the server. Below are the types that can be part of our schema:

- **Scalar**: Int, Float, String, Boolean, ID. The ID type is a unique identifier used to re-fetch an object or as the key for a cache.
- **Object**: This is a complex type with its own characteristic. For example, an Author object that has first name, last name, rating and number of courses authored.
- **Enum**: They are special scalar types that are restricted to a particular set of allowed values.
- **Query**: Queries specify the data we want to fetch from a GraphQL server.
- **Mutation**: This is for adding, updating or deleting data from the API. Query and Mutation types act as an entry point into the schema and are the same as any other GraphQL object type.
- **Non-Nullable**: By default, each of the scalar type can be set to null. To override this behavior and ensure that the field cannot be null, we add an exclamation mark. In the below example we have declared id and list of courses as a non-nullable fields.

```graphql
type Author{
    id: ID!
    firstName: String
    lastName: String
    rating: Float
    numOfCourses: Int
    courses: [String!]
}
```
<br>

## Maven Dependencies
To include GraphQL in your project, a couple of dependencies are needed: 
-  **graphql-spring-boot-starter**: It will add and automatically configure a GraphQL Servlet that you can access at `/graphql` . This starter will also use a GraphQL schema library to parse all schema files found on the classpath. The starter will also set up an endpoint that can access POST requests.
- **graphql-java-tools**: A helper library to parse the GraphQL schema. 
- **graphiql-spring-boot-starter**: Although optional but extremely useful, GraphiQL is an in-browser IDE for writing, validating and testing GraphQL queries. Supports live syntax and validation errors. GraphiQL is a simple web app that is able to communicate with any GraphQL Server and execute queries and mutations against it.

Note: These dependencies need to be "manually" added to the pom.xml file.

```xml
<dependency>
    <groupId>com.graphql-java</groupId>
    <artifactId>graphql-spring-boot-starter</artifactId>
    <version>5.0.2</version>
</dependency>
<dependency>
    <groupId>com.graphql-java</groupId>
    <artifactId>graphql-java-tools</artifactId>
    <version>5.2.4</version>
</dependency>
<dependency>
    <groupId>com.graphql-java</groupId>
    <artifactId>graphiql-spring-boot-starter</artifactId>
    <version>5.0.2</version>
</dependency>
```
<br>

In our `application.properties` file:
- Enable H2, GraphQL and GraphiQL
- Note: `graphql.servlet.mapping` and `graphiql.endpoint` values ***do need to match***, as that is how GraphQL and GraphiQL will interact.

```properties
spring.h2.console.enabled=true
spring.h2.console.path=/h2
spring.datasource.url=jdbc:h2:mem:dbname

graphql.servlet.mapping=/graphql
graphql.servlet.enabled=true
graphql.servlet.corsEnabled=true

graphiql.enabled=true
graphiql.endpoint=/graphql
graphiql.mapping=graphiql
```
<br>
## Building a GraphQL API
**Step 1: Add dependencies**: Add Spring Web, JPA, H2, GraphQL and GraphiQL dependencies. <br>
**Step 2: Data**: We will store the below location data in an in-memory H2 database. Create a `data.sql` file under `/src/main/resources` with the below input:

```sql
INSERT INTO location (id, name, address) VALUES (1, 'ATL Airport','Atlanta airport location');
INSERT INTO location (id, name, address) VALUES (2, 'MIA Airport','Miami airport location');
INSERT INTO location (id, name, address) VALUES (3, 'LAX Airport','Los Angeles airport location');
INSERT INTO location (id, name, address) VALUES (4, 'PHX Airport','Phoenix airport location');
INSERT INTO location (id, name, address) VALUES (5, 'ORD Airport','Chicago airport location');
```
<br>
**Step 3: Define GraphQL Schema**: Create a `location.graphqls` file under `/src/main/resources` - this schema defines all Query and Mutation operations for the Location type:

```graphql
type Location {
 id: ID!
 name: String!
 address: String!
}

type Query {
 findAllLocations: [Location]!
}

type Mutation {
 newLocation(name: String!, address: String) : Location!
 deleteLocation(id:ID!) : Boolean
 updateLocationName(newName: String!, id:ID!) : Location!
}
```
<br>
**Step 4: Create the Entity**: Every complex type (like Location type above) in the GraphQL schema will be represented by a Java class, below is our `Location.java` class. Note that the schema type and Java class name should exactly match:

```java
@Entity
public class Location {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String address;

    public Location(Long id, String name, String address) {
        this.id = id;
        this.name = name;
        this.address = address;
    }

    public Location(String name, String address) {
        this.name = name;
        this.address = address;
    }

    public Location() {}

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }
```
<br>

**Step 5: Create Repository**: Like any RESTful API, we will create a repository:
```java
@Repository
public class LocationRepository extends CrudRepository<Location, Long>{

}
```
<br>

**Step 5: Create Query Resolvers**: Create a `Query.java` class which impelments the `GraphQLQueryResolver` - this allows Spring to automatically detect and call the right method and respond to the GraphQL queries declared within the schema. In the schema, we have defined one query, findAllLocations() - this will mapped to the findAllLocations() in the Query class.

```java
import com.coxautodev.graphql.tools.GraphQLQueryResolver;
import org.springframework.stereotype.Component;

@Component
public class Query implements GraphQLQueryResolver {
    private LocationRepository locationRepository;

    public Query(LocationRepository locationRepository) {
        this.locationRepository = locationRepository;
    }

    public Iterable<Location> findAllLocations() {
        return locationRepository.findAll();
    }
}
```
<br>

**Step 6: Create Mutators**: Create `Mutation.java` class which implements `GraphQLMutationResolver` - this allows Spring to automatically detect and call the right method and respond to the GraphQL mutations declared within the schema. Our schema has 3 mutations defined to create, update and delete locations - these mutations will mapped to the methods defined in the Mutation class.
For the update method, if the location we are trying to update does not exist, it will throw a LocationNotFoundException custom exception discussed in the next step.

```java
import com.coxautodev.graphql.tools.GraphQLMutationResolver;
import org.springframework.stereotype.Component;

@Component
public class Mutation implements GraphQLMutationResolver {
    private LocationRepository locationRepository;

    public Mutation(LocationRepository locationRepository) {
        this.locationRepository = locationRepository;
    }

    public Location newLocation(String name, String address) {
        Location location = new Location(name, address);
        locationRepository.save(location);
        return location;
    }

    public boolean deleteLocation(Long id) {
        locationRepository.deleteById(id);
        return true;
    }

    public Location updateLocationName(String newName, Long id) {
        Optional<Location> optionalLocation =
                locationRepository.findById(id);

        if(optionalLocation.isPresent()) {
            Location location = optionalLocation.get();
            location.setName(newName);
            locationRepository.save(location);
            return location;
        } else {
            throw new LocationNotFoundException("Location Not Found", id);
        }
    }
}
```
<br>

**Step 7: Handle Exceptions**: Create a `LocationNotFoundException.java` class that extends `RuntimeException` and  implements `GraphQLError`. GraphQLError provies a field called `extensions` which allows you to provide additional data to the error object and is sent back to the client - in our case we are passing the id of the invalid location, invalidLocationId.:

```java
import graphql.ErrorType;
import graphql.GraphQLError;
import graphql.language.SourceLocation;

import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class LocationNotFoundException extends RuntimeException implements GraphQLError {

    private Map<String, Object> extensions = new HashMap<>();

    public LocationNotFoundException(String message, Long invalidLocationId) {
        super(message);
        extensions.put("invalidLocationId", invalidLocationId);
    }

    @Override
    public List<SourceLocation> getLocations() {
        return null;
    }

    @Override
    public Map<String, Object> getExtensions() {
        return extensions;
    }

    @Override
    public ErrorType getErrorType() {
        return ErrorType.DataFetchingException;
    }
```
<br>

**Step 8: Run your application**: 
- Navigate to `localhost:8080/graphql/schema.json` - This will show all of the schemas on the GraphQL server.
- If you want to use **Postman** to execute queries:
	- Select `POST` as the request type to `localhost:8080/graphql` URL
	- Header: Set `Content-Type` to `application/json`
	- Body: List the query you want to execute, `"query": "findAllLocations {name id address }"`
	- Click Send to view the output (list of locations)
- If you want to use **GraphiQL** to execute queries:
	- Visit `localhost:8080/graphiql` in the browser
	- Enter the queries and mutations in the command window and click the Execute button to view the result. In the below example, we are executing the mutation to create a new location in the database and we are requesting the id, name and address back:

```graphql
mutation{
	newLocation(
		name: "JFK Airport",
		address: "New York, New York airport location"){
			id
			name
			address
		}
}
```
<br>

In the below mutation query, we can see how to delete an entry:

```graphql
mutation{
	deleteLocation(id:1)
}
```
<br>
