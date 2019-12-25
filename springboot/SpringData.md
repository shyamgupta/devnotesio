---
layout: springboot
permalink: /springboot/SpringData/
---
## Map a Database Table to a Java Class
Let's start with a simple example, where we map one database table to one JPA entity: below we have a STUDENT table with 4 columns:

```sql
student_id INT(11)
student_name VARCHAR(45)
student_fulltime TINYINT(1)
student_age INT(11)
```
<br>

Below we have a Java Student class with 4 attributes that represents the above database table. 
- The class must be annotated with `@Entity`. 
- Entities must have a unique identfier annotated with `@Id`. JPA will generate the id when persisted, for this reason we don't include student_id in the constructor. The id field is used to hold the database identity of each stored entity—the primary key when using a relational database. Here, we are delegating the responsibility to generate unique values of the identity field to Spring Data.
- `@Column` maps the class atributes to table columns.

```java
@Entity
@Table(name="STUDENT")
public class Student{
	@Id
	@GeneratedValue(strategy=GenerationType.IDENTITY)
	@Column(name="student_id")
	private Integer student Id;

	@Column(name="student_name")
	private String name

	@Column(name="student_fulltime")
	private boolean fullTime;

	@Column(name="student_age")
	private Integer age;

	public Student (String name, boolean fullTime, Integer age){
		this.name = name;
		this.fullTime = fullTime;
		this.age = age;
	}

	//getters and setters logic here
}
```
<br>

Now that we have define dthe object to relational mapping metadata, all of our database related coding can stay in the logical world because JPA will take care of the physical world for us.

## Repository

The most effective way for our application to interact with this table is using the full force of Spring Data JPA. To do this, we need to create a repository interface for the Student class:

```java
public interface StudentRepository extends CrudRepository<Student, Integer> {}
```
<br>

The preceding interface might appear empty, but the inherited methods inside CrudRepository provide the core CRUD operations we need, including save, findOne, findAll, delete, exists, and count. The interface is generically typed to match up with the domain class and the ID's field type.

With this code in place, when Spring Boot creates an application context, Spring Data JPA will scan and discover our repository definition. Then it will automatically generate a concrete proxy that implements this interface. This saves us the labor of writing all these queries. Below are some useful methods that CrudRepository includes:


<table class="table table-striped table-bordered">
	<thead class="thead-dark">
		<tr>
			<th>Method</th>
			<th>Description</th>
		</tr>	
	</thead>
	<tbody>
		<tr>
			<td><pre>count()</pre></td>
			<td>Returns the number of entities available</td>
		</tr>
		<tr>
			<td><pre>delete (ID id)</pre></td>
			<td>Deletes the entity with the given id</td>
		</tr>
		<tr>
			<td><pre>deleteAll()</pre></td>
			<td>Deletes all entities managed by the repository</td>
		</tr>
		<tr>
			<td><pre>exists(ID id)</pre></td>
			<td>Returns whether an entity with the given id exists</td>
		</tr>
		<tr>
			<td><pre>findAll()</pre></td>
			<td>Returns all instances of the type with the given IDs</td>
		</tr>
		<tr>
			<td><pre>findOne(ID id)</pre></td>
			<td>Retrieves an entity by its id </td>
		</tr>
		<tr>
			<td> <pre>save(S entity)</pre></td>
			<td>Saves a given entity</td>
		</tr>
	</tbody>
</table>
<br>





