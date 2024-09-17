
If you want to add logging to a class in Spring you first have to had the following line in it.

```java
private static final Logger log = LoggerFactory.getLogger(ExampleController.class);
```

Then you put the logging command to your method. Which logs the message when the method is ran.

```java
@GetMapping("/allexample")
  public List<Example> getAllExamples() {
    log.info("All examples listed.")
    return exrep.findAll();
  }
```

There is five different kinds of log messages

- Error
- Warning
- Info
- Debug
- Trace

If you want to change what kind of logging messages you can see you have to add to the application.properties file.

```
logging.level.root=Error  //will only show Error logging.
```


