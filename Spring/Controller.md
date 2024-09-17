Spring adheres to the [MVC](https://developer.mozilla.org/en-US/docs/Glossary/MVC) pattern.

**Controller** classes usually take care of the logic within this pattern. What the controller classes also do is responding to requests, which is their main task in Spring.
###### Routing
Decides which URL should respond to which method/function. This is pretty easy in Spring. You just add the route in an annotation over the method that will respond to that route.

![[controllerannotation.png]]

**@RestController** typically used along with the request mapping annotation. Needs to be added for a class to be able to respond to URLs. The annotation is a specialised one that includes the controller and response body annotations. Every request handling method of the class with the annotation serialise return objects into HttpResponse.  ^8f3296

```java
@RestController
public class Example {
  stuff
}
```

***

**@RequestMapping** used to route a method to a certain URL, you use a parenthesis were you put the string of the path. Can't be used on different methods mapping to the same URL. ^24db79

```java
@RestController
public class Example {

  @RequestMapping("/sometext") // Path for the URL.
  public String someText() {
    return "Some text...";
  }
}
```

***

**@RequestParam** added to the request mapping method if you want to use parameters. If a parameter been added to the method but not used the site will return a 404. You can add default values and optional values though to the parameters. To use the parameter in the URL you add '?' then the name of the variable '=' the data you want. For multiple ones you use a '&' between them. You can also have lists as parameters. When you put the parameters into the URL they don't have to be in the same order as in the code. If you use \** the path will be a wildcard that covers all paths and if you combine it with for example a response status annotation you can have a mapping for all 404 requests. ^0f56fe

These are called query parameters.

```java
// Single param
// example URL localhost:8080/singleparam?name=Joeyjojojrshabadoo

@RequestMapping("/singleparam")
public String singleParam(@RequestParam String name) {
  return "Hello " + name;
}

// Multi param
// example URL localhost:8008/multiparm?name=JJJJr&sName=Shabadoo

@RequestMapping("/multiparam")
public String multiParam(@RequestParam String name, @RequestParam String sName) {
  return "Hello " + name + " " + sName;
}

// Default param

@RequestMapping("/defaultparam")
public String defaultParam(@RequestParam(defaultValue="JJJJr") String name) {
  return "Hello " + name;
}

// Optional param
// If nothing is added will return null

@RequestMapping("/optionalparam")
public String optionalParam(@RequestParam(required=false) String name) {
  if (name == null) {
    name = "";
  }
  return "Hello " + name;
}

// List param
// There are two different ways to write the list in the URL
// Method 1: localhost:8080/listparam?list="Joey","JoJo","Jr"
// Method 2: localhost:8080/listparam?list="Joey"&list="JoJo"&list="Jr"

@RequestMapping("/listparam")
public String listParam(@RequestParam List<String> list) {
  return "Hello " + list;
}

// Wildcard combined with response status to show error message for 404.

@RequestMapping("/**")
@ResponseStatus(HttpStatus.NOT_FOUND)
public String error404() {
  return "Resource not found";
}

```

***

**@PathVariable** unlike the query parameters is added directly in the path of the URL. You therefore have to add the parameter in the request mapping path using {} to enclose it. Then like the query parameters you add the annotation in the method. The parameter doesn't have to be the last thing in the path it can be in bedded between other stuff. ^6db3ed

```java
// localhost:8080/pathvariable/JoeyJoJoJrShabadoo
// -> Hello JoeyJoJoJrShabadoo

@RequestMapping("/pathvariable/{name}")
public String pathVariable(@PathVariable String name) {
  return "Hello " + name;
}
```

***

**@PostMapping** like the request mapping but for POST command. It is used in with the annotation **@RequestBody** were you just like indicated by name specify what will be sent with the body. You can add an object directly like the example below. ^acd090

```java
@RestController
public class Example {

  @PostMapping("/sometext") 
  public void someText(@RequestBody SomeObject example) {
    SomeJSONList.add(example);
  }
}
```

The question after setting this up is how to send the POST. It can be done in different ways, if you want to do it via the terminal you can use **curl**. If you want to do it with a GUI you can use software like Postman or Insomnia.

A curl command posting JSON data can look like this. You don't need to use the -X flag if you're using POST as that is the default. If you want to do some of the others you should add it e.g -X GET.

```terminal
curl URLpath -d '{"id": 8, "name": "JJJJr"}' -H 'Content-Type: application/JSON'

// -d is the Data you send.
// -H is the Header.
```

***

**@PutMapping** the routing for PUT. Works like the post mapping annotation. If using curl remember to add the -X PUT. ^6a97c1

```java 
@PutMapping("friends/update")
  public void updateExample(@RequestBody Example example) {
    Example toUpdate = exampleList.stream()
    .filter(e -> example.getId() ==   e.getId()).findFirst().orElse(null);
        if (toUpdate == null) {
            exampleList.add(example);
        } else {
            toUpdate.setName(example.getName());
        }
    }
```

***

**@GetMapping** is mapping a GET request to a method or class. Used for [[REST#^e46a60|HATEOAS]]. ^8d52a9

[Tutorial getting started with REST](https://spring.io/guides/tutorials/rest)
Note that a dependency needs adding. (Spring Boot Starter HATEOAS in this case)

Example HATEOAS code. In current Spring version it's important to import static the mvc link builder (added to example below).

```java
import static org.springframework.hateoas.server.mvc.WebMvcLinkBuilder.*;

@GetMapping("/example/{id}")  
EntityModel<Friend> one(@PathVariable int id) {  
  Example example = exampleList.stream()
  .filter(e ->  e.getId() == id).findFirst().orElse(null);  
  return EntityModel.of(example, linkTo(methodOn(ExampleController.class).one(id)).withSelfRel(),        linkTo(methodOn(ExampleController.class).allExample()).withRel("examples"));  
}  
  
@GetMapping("/examples")  
CollectionModel<EntityModel<Example>> all() {  
  List<EntityModel<Example>> examples = exampleList.stream()
  .map(e -> EntityModel.of(e,  
  linkTo(methodOn(ExampleController.class).one(e.getId())).withSelfRel(),
linkTo(methodOn(ExampleController.class).all()).withRel("examples"))).toList();  
  return CollectionModel.of(examples, linkTo(methodOn(ExampleController.class).all()).withSelfRel());  
}
```