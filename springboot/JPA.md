---
layout: springboot
permalink: /springboot/JPA/
---
## Persistence in Java: JPA
The Java Persistence API (JPA) specification and it's implementations help developers in accessing, persisting and managing data between Java objects, classes and relational databases. Object Relational Mapping (ORM) connect Java and relational db's. 

Relational databases store data in rows and columns, but Object-Oriented languages like Java does not know about rows and columns. Object Oriented languages organize data in objects, classes,packages, interfaces etc. and relational db's know nothing about these Object Oriented concepts. ORM bridges this gap bewteen Object Oriented Languages and Relational Databases - it allows objects to be mapped to tables and tables can be mapped to objects. 

The process of ORM is often delegated to external tools or frameworks. In Java EE this framework is called JPA, which is a specification for accessing, persisting and managing data between Java objects, classes and relational databases. 

Using JPA, we can access a database without writing any SQL code. JPA is an abstraction over JDBC (Java Database Connectivity). 

## Hibernate

JPA lives in the `javax.persistence` package, with different sub-packages, but most of the core API and annotations live in javax.persistence. Since JPA is a specification (set of empty methods and collection of interfaces) or a set of guidelines, it cannot do anything on its own. In order to be fullly functional, JPA needs an implementation, also known as an instance provider. Some of the JPA implementations are:

1. EclipseLink (Oracle)
2. Hibernate (JBOSS and RedHat): Spring with JPA uses Hibernate as the persistence provider.
3. OpenJPA (IBM)
4. Data Nucleus (JPOX)

## JPA Metadata

The mapping between Java objects and database tables is defined via the persistence metadata, which is used by the JPA instance provider to perform correct database operations. There are two ways to define JPA metadata:

1. **Annotations**: Using annotations in the Java class, @Entity, @Id, @Table, @Column etc. is the most common way and is what we will use. Spring can scan all packages looking for entities described by the @Entity annotation.
2. **XML**: Alternatively, metadata can be defined in XML or a combination of XML and Annotations. 

JPA can be setup with no XML, which is very convenient.

## Entities

Entities are objects that live in a database and have the ability to be mapped to a database. They are defined by the `@Entity` annotation, and support inheritence, relationships etc and also have a unique identifier. 

Once the entity has the correct mapping metadata, JPA allows us to query the entity in an object-oriented way. The entity manager, `javax.persistence.EntityManager` is responsible for orchestrating this process - it performs CRUD operatations, allowing is to persist, update, retrieve, and delete objects from a database. Enttity Manager API allows us to interact with the database without writing any JDBC or SQL code.

JPA also comes with a more powerful language, **Java Persistence Query Language (JPQL)** - allowing us to retrieve data with an object-oriented query language. JPQL is a layer on top of SQL. It's syntax is similar to SQL, but refers to entities and attributes instead of tables and columns.

1. Note that ***results from a JPQL query are returned as a collection of entities***.
2. JPQL does NOT support schemaless or NoSQL databases. It also does not support JSON.

Now, let's see how to map a database table to a Java class: we've one database table (STUDENT) with 4 columns and we need to map it to one JPA entity:

```sql
-- Table name: STUDENT
student_id INT(11) <-- PRIMARY_KEY
student_name VARCHAR(45)
student_fulltime TINYINT(1)
student_age INT(11)
```
<br>
```java
@Entity
@Table(name = "STUDENT")
public class Student {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "student_id")
    private Integer studentId;
    
    @Column(name = "student_name")
    private String name;
    
    @Column(name = "student_fulltime")
    private boolean fullTime;
    
    @Column(name = "student_age")
    private Integer age;

    public Student(String name, boolean fullTime, Integer age){
        this.name = name;
        this.fullTime = fullTime;
        this.age = age;
    }
```
<br>

As you can see, the 4 database table columns have been mapped to a Java class with four attributes.

- **@Entity** annotation defines that a class can be mapped to a database table.
- **@Table** defines the table that the entity is mapped to.
- **@Id** marks a field as a primary key field. Entites must have this unique identifier field.
- **@GeneratedValue** generates a unique identifier for the entity, that's why we don't have the id attribute in the constructor as JPA will generate it.
- **@Column** changes the name or length of any column in the database.

Now that we have defined the object to relational mapping metadata, all of our database related coding can stay in the logical world, because JPA will take care of the physical world for us.

#### Mapping multiple tables to a Java class

Continuing with the previous example, below is an ER diagram with multiple tables:

<img src="/assets/img/er.png" alt="ER diagram" class="img-fluid img-thumbnail">

There are three new tables in our database:DEPARTMENT, COURSE and Enrollment. Students are enrolled in courses, so we have a join table called Enrollment. Lets see how this physical model can be mapped to a logical model.

We jave two new entities, COURSE and DEPARTMENT

```java
@Entity
@Table(name = "Course")
public class Course {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "course_id")
    private Integer id;
    
    @Column(name = "course_name")
    private  String name;
    
    @ManyToOne
    @JoinColumn(name = "course_dept_id")
    private Department department;
    
    public Course(String name,Department department){
        this.name = name;
        this.department = department;
    }  
}
```
<br>

```java
@Entity
@Table(name = "Department")
public class Department {
    @Id
    @GeneratedValue
    @Column(name = "dept_id")
    private Integer id;

    @Column(name = "dept_name")
    private String name;

    @OneToMany(mappedBy = "department",fetch = FetchType.EAGER,cascade = CascadeType.ALL)
    private List<Course> courses = new ArrayList<>();

    public Department(String name){
        this.name = name;
    }

}
```
<br>

Let's see how we are handling Foreign Key:

- The course table has a `course_dept_id` column. The `@JoinColumn` annotation declares this physical mapping between the Course and Department entity. 
- The `@ManyToOne` annotation shows cardinality - many courses are mapped to one department.
- The Department entity has a list of courses that annotates with `@OneToMany`: one department has many courses. The `mappedBy="department"` parameter refers to the department attribute in the Course class. By default, OneToMany attributes are not automatically fetched from the database - this is callled Lazy Loading. The `FetchType.EAGER` setting overrides this default behavior, so when a department is ready from the database, JPA also populates the associated courses. The `cascade` parameter controls the state of the collection attribute - with `CascadeType.ALL`, any state changes to the department, should apply to its courses as well.


Now, let's look at the Enrollment table - notice the @OneToMany annotation where we are specifying the join table:

```java
@Entity
@Table(name = "STUDENT")
public class Student {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "student_id")
    private Integer studentId;

    @Column(name = "student_name")
    private String name;

    @Column(name = "student_fulltime")
    private boolean fullTime;

    @Column(name = "student_age")
    private Integer age;

    @OneToMany(fetch = FetchType.EAGER,cascade = CascadeType.ALL)
    @JoinTable(name = "Enrollment",joinColumns = {@JoinColumn(name = "student_id")},
        inverseJoinColumns ={@JoinColumn(name="course_id")})
    private List<Course> courses = new ArrayList<>();


    public Student(String name, boolean fullTime, Integer age){
        this.name = name;
        this.fullTime = fullTime;
        this.age = age;
    }

    //getters
}
```




































