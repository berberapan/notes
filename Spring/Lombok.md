A library that can be used in Spring. 

Adds implicit getters and setters to POJO-classes. Also adds default constructor.

**@Data** bundled annotation that bundles features from other annotations in the library that creates among other things toString, getter, setter and args constructor. Which means you don't have to write any of these yourself just the class and it's variables. ^7ffa30

The args constructor that the annotation is bundled with is **@RequiredArgsConstructor** which means that it won't add a constructor to all fields unless the variable is final or got a **@NonNull** annotation. If you want to change the access level you have you have to add the annotation in the same way as all args below.  ^bce00b

```java
import lombok.Data;
import lombok.NonNull;

@Data
public class somePOJO {
  final int someNum;
  @NonNull String someText;
  final long someLongNum;
}
```

If you want all fields added to the constructor you need to use **@AllArgsConstructor**. You can still use the non null annotation here, it will make null checks. You can set the access level of the variables in the annotation by using (access = AcessLevel._LEVELOFACCESS_) otherwise it defaults to public. ^7fd7ff

```java
import lombok.Data;
import lombok.AllArgsConstructor;
import lombok.AccessLevel;

@Data
@AllArgsConstructor(access = AccessLevel.PROTECTED)
public class somePojo {
  int someNum;
  String someText;
  long someLongNum;
}
```

Use of these annotations doesn't make a conflict if you create your own generators. They can co-exist, so you can make the boilerplate constructor with Lombok and then make your own specialised ones.