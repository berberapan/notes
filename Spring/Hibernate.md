Hibernate is a Java framework that implements the **JPA** (Java Persistence API) specification. It allows you to manage data persistence by mapping your Java objects to relational database tables and automatically generating the required SQL. It's an ORM.

**Spring Data JPA** library needs to be added for everything to work.

***

**@Entity** is used when a POJO is supposed to match a table in the database. It needs to be added before the declaration of the class. ^7d56c4

```java
@Entity
public class Example {...}
```

***

**@Table** is used when you want to map a class to an already existing table. This is mostly used if you're working in a Database First manner. Otherwise it's mostly used if you want the table to have a different name to the class. You set the name with a name parameter. ^5d8a1a

```java
@Entity
@Table(name="Examples")
public class Example {...}
```

***

**@Id** is required in an entity. You set the primary key with the annotation. The class variable you add it for needs to be of the data type Long. ^700f55

```java
@Entity
public class Example {
  
  @Id
  private Long id;
}
```

***

**@GeneratedValue** used together with Id. Auto-increment ids to guarantee they are unique. ^7572bd

```java
@Entity
public class Example {
  
  @Id
  @GeneratedValue
  private Long id;
}
```

***

**@Column** works like table but is used for the variables. Mostly used in Database First but can also be used if you want a different name on the column in the database. ^be05e1

```java
@Entity
public class Example {
  
  @Id
  @GeneratedValue
  private Long id;

  @Column(name = "name")
  private String exempelName;  
}
```

***
##### Application.properties

Example of settings. 

![[applicationpropertieshibernate.png]]

Ddl-auto got several possible settings. Update just updates the current database. Create-drop drops the database after use and creates a new one upon starting the application.

***
##### Repository

The normal way to use the POJO you're making a table of is to create a repository. It's an interface that comes with several of the standard SQL queries when extending the JPArepository.

```java
public interface ExampleRepo extends JpaRepository<Example, Long> {}

// You add your Pojo class as data type and Long that corresponds to your primary key.
```

You can then call on the repository in your control class even though it's an interface.

```java
@RestController
public class ExampleController {
  private final ExampleRepo exrep;

  public ExampleController(ExampleRepo exrep) {
    this.exrep = exrep;
  }

  @GetMapping("/allexample")
  public List<Example> getAllExamples() {
    return exrep.findAll();
  }
}
```

It got several CRUD methods. In the example above findAll() is used that returns an iterable.
You can find the others in the [documentation](https://docs.spring.io/spring-data/jpa/docs/current/api/org/springframework/data/jpa/repository/JpaRepository.html)

***
##### Bootstrapping

When bootstrapping in Spring you use **Beans**. A bean can mean a lot of different things depending on who you talk to but in Spring it usually means an initialised object of a class that we have access to, ergo a POJO.

To bootstrap an application in Spring it's easy to create a method using **CommandLineRunner** and annotate it with **@Bean**. This needs to be done in the main application class. ^61f3c5

When creating the new example the POJO need a constructor without the Id creation to make it easier.

```java
@Bean
public CommandLineRunner boot(ExampleRepo exrep) {
  return (args) -> {
    exrep.save(new Example("ex1", "param"))
    exrep.save(new Example("ex2", "param"))
    exrep.save(new Example("ex3", "param"))
  }
}
```

***
##### Customised queries

**@Query** if you want to make more advanced queries and not just use the default JpaRepository ones you need to use this annotation. You put the SQL query in parenthesis. ^e85852

```java
public interface ExampleRepo extends JpaRepository<Example, Long> {

  @Query("SELECT text from Example order by text")
  public List<String> getTextOrder();
}
```

***

If you're going to update data you should use the following annotations **@Modifying** and **@Transactional**. ^a763d9

```java
@Modifying
@Transactional
@Query("update Example set text=:newText where text=:oldText")
public void updateText(String oldText, String newText);
```

***

**DTO** (Data Transfer Object) is used if we want to make some type of grouping, e.g. a count for a value in a column.

So you first make a DTO object, it should not have the entity annotation. Create the class with the values you're looking for. If you want a count for example you give the class a variable of the what you're grouping it on and a variable for the value of the count (int or long)

```java 
public class ExampleDTO {
  private String exampleGroup;
  private Long counter;
}
```

In the repository you then add the full path to the DTO class in the query when creating a new instance.

```java 
@Query(value= "SELECT new fullpath.ExampleDTO(exampleGroup, count(counter)) " +
	  "FROM Example group by text")
public List<ExampleDTO> getCounter();
```

***
##### 1:1

**@OneToOne** is used to make a one to one relationship between two entities in Hibernate. It's used with **@JoinColumn** to create the foreign key. You put the annotations in the entity were you want the FK in. ^476dfa

```java
@Entity
public class Example {
  @Id
  @GeneratedValue
  private Long id;
  private String name;
  @OneToOne (cascade = CascadeType.ALL)
  @JoinColumn
  private ExampleNum exampleNum;
}
```

You can add **cascade** as a variable to the relation annotation. Can for example have a method deleting the instance of an entity and it will remove the correlating entry in the other table.

Also works when creating instances so  you don't have to save the corresponding entry first.
***
##### N:1

**@ManyToOne** is used for the N:1 relationship. You should use the JoinColumn for this one as it needs a foreign key, but avoid using cascade as you don't want a delete of an entity in this relationship to remove one of the many's that can be tied to several different entries. ^e3182b

```java
public class Example {
  @Id
  @GeneratedValue
  private Long id;
  private String name;
  @ManyToOne
  @JoinColumn
  private Category category;
}
```

***
##### N:M

**@ManyToMany** is used for the N:M relationship. To get the join table you use the annotation **@JoinTable**. Cascade more reasonable to use here. ^c4f584

```java
public class Example {
  @Id
  @GeneratedValue
  private Long id;
  private String name;

  @ManyToMany
  @JoinTable
  private List<JoinExample> je = new ArrayList<>();
}
```

***
##### 1:N

**@OneToMany** is the annotation for the 1:N relationship. In this table you won't have a foreign key so you have to tell Hibernate what column you represent in the other table so you use the **mappedBy** parameter. It is a bidirectional mapping. ^77fc6f

When doing this it will create a circular reference. A circular reference can happen in 1:1 and N:M too. The easiest way to solve a circular reference is to use the **@JsonIgnore** annotation. You use the annotation in one of the classes that is involved in the circular ref, it then breaks the loop so the objects won't get be printed. Put it where it makes the most sense depending on what you want to display. ^e07b6f

```java
public class Example {
  @Id
  @GeneratedValue
  private Long id;
  private String name;
  @OneToMany(mappedBy="ManyExample")
  @JsonIgnore
  private List<Category> category;
}
```