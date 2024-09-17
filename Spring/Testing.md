[Tutorial for testing in Spring](https://spring.io/guides/gs/testing-web)

Spring is such a complex framework so you need to test how your code integrates with it along with testing your own code as in a unit test.

When you start up a project it starts up with an application test. It will just be a test checking if the Spring project can start up.

The main annotation to use when creating a test is the **@SpringBootTest**, it tells Spring Boot to check for the **@SpringBootApplication** and use it to start up a spring application context. ^b30a64
##### Testing of controllers

The models and repositories don't need much testing but the controllers often got lots of things worth testing. They can be tested in a couple of different ways.

- Test that the controller-object actually starts up.
- Test that correct method responds to the correct routing (the URL).
- Test that correct data is returned.

Example of test checking if the controller object is starting up.

```java
@SpringBootTest
public class ControllerCreationTest {

  @Autowired
  private ExampleController controller;

  @Test
  public void contextLoads() throws Exception {
    assertThat(controller).isNotNull();
  }
}
```

The **@Autowired** annotation creates an instance of the controller in this example but can be used for other types as well. So you don't need to put in the new keyword, spring sort it for you. It injects the object.  ^eb5750

The actual test gets annotated with **@Test** and that's where you put you're asserts etc. ^c95a45

If you want to test if a controller method is responding to an URL it should respond to you will need to add a bit more code. It's more of an integration test.

```java
@SpringBootTest(webEnviroment = SpringBootTest.WebEnviroment.RANDOM_PORT)
public class ControllerRespondTest {

  @Value(value = "${local.server.port}")
  private int port;

  @Autowired
  private TestRestTemplate restTemplate;

  @Test
  public void example() throws Exception {
    assertThat(this.restTemplate.getForObject("http://localhost:" + port + "/",
    String.class)).contains("Exempeltext");
  }
}
```

In this test you connect to the web server and use a random port to avoid any issues with the ports in use. **@Value** is used to set the local server port value to a variable. TestRestTemplate is a special object that helps handle the response. In the test you can see that the resttemplate is using a get for the URL to receive a String. ^e2cc29

There are other more lightweight ways of testing the controllers without having to start the web server with all that entails. Example below shows a mock version of the previous example. Uses **@AutoConfigureMockMvc** with a MockMvc object. ^5a3815

```java
@SpringBootTest
@AutoConfigureMockMvc
public class ControllerRespondTest {

  @Autowired
  private MockMvc mockMvc;

  @Test
  public void example() throws Exception {
    this.mockMvc.perform(get("/")).andDo(print()).endExpect(status().isOk())
      .andExpect(content().string(containsString("Exempeltext")));
  }

// Test for URL not exsisting.

  @Test
  public void error404() throws Exception {
    this.mockMvc.perform(get("/incorrectUrl")).andDo(print())
      .andExpect(status().isNotFound());
  }
}
```

You can do it even more lightweight using the annotation **@WebMvcTest**. You test the controller methods without starting the spring context. ^ddf416

```java
@WebMvcTest
public class ControllerRespondTest {

  @Autowired
  private MockMvc mockMvc;

  @Test
  public void example() throws Exception {
    this.mockMvc.perform(get("/")).andDo(print()).endExpect(status().isOk())
      .andExpect(content().string(containsString("Exempeltext")));
  }
```

Often when testing in Spring environment you can't have static tests as you need to test with databases that changes and get new info. You can't test things like delete with a production database, so instead you create a mock again with mock objects.

Below is a test of the common CRUD methods used in Spring. We use **@MockBean** to create our mock repository and **@BeforeEach** to create objects and set the mock repository behaviour before each test. ^1f2704

In the example below when using the findAll json is used instead of string. String can be used but it's much more handy to use json as you don't have to worry about the order of the values in each entry.

```java
@SpringBootTest
@AutoConfigureMockMvc
class ExampleControllerTest {

  @Autowired
  private MockMvc mvc;
  
  @MockBean
  private ExampleRepository mockRepo;

  //Creates objects and mockRepo behaviour before every test.
  @BeforeEach
  public void init() {
    Example e1 = new Example(1L, "num1");
    Example e2 = new Example(2L, "num2");

    when(mockRepo.findById(1L)).thenReturn(Optional.of(e1));
    when(mockRepo.findById(2L)).thenReturn(Optional.of(e2));
    when(mockRepo.findAll()).thenReturn(Arrays.asList(e1, e2));
  }
  //Create test
  @Test
  void addExample() throws Exception {
    this.mvc.perform(get("/add?num=num3")).andExpect(status().isOk())
      .andExpect(content().string(equalTo("Example added")));
  }
  //Delete test
  @Test
  void deleteExample() throws Exception {
    this.mvc.perform(get("/delete/1")).andExpect(status().isOk())
      .andExpect(content().string(equalTo("Example 1 deleted")));
  }
  // Get all test. 
  @Test
  void getAllExample() throws Exception {
    this.mvc.perform(get("/examples")).andExpect(status().isOk())
      .andExpect(content().json("[{\"id\":\"1\", \"num\":\"num1\"},
        {\"id\":\"2\", \"num\":\"num2\"}]"));
  }
  // Get object by id test. 
  @Test
  void getExampleById() throws Exception {
    this.mvc.perform(get("/example?id=1")).andExpect(status().isOk())
      .andExpect(content().json("[{\"id\":\"1\", \"num\":\"num1\"}]"));
  }
  //Post test
  @Test
  void addExample() throws Exception {
    this.mvc.perform(post("/addByPost").contentType(MediaType.APPLICATON_JSON)
      .content("{\"id\": 4, \"num\": \"num4\"}"))
      .andExpect(status().isOk())
      .andExpect(content().string(equalTo("Example added")));
}
```