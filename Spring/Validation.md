You can set validation for you class variables. Constraints are imported from jakarta.validation.

**@Valid** needs added to the Spring method you want the validation for. In the example below any addition needs to match the validation set in the Example class to create an example object. ^9efa50

```java
@RequestMapping("example/add")
public void addExample(@Valid @RequestBody Example example)
  code to add object
```

***

##### Class validations

**@NotNull** works for all data types, puts a constraint on the variable that it can't be Null. ^2940b7

***

**@NotEmpty** for strings, arrays and such. Checks so variable isn't empty. ^f5d5c6

***

**@Size** if this annotation is used you can set minimum and max size of the element. Works for collections and strings. You set the minimum/maximum with the min and max params. ^3aeb8d

```java
@Size(min=2, max=10, message="Follow the rules!")
String example;
```

***

For all the above annotations you can set an error message with the **message** parameter.

[Documentation for all available constraints](https://jakarta.ee/specifications/bean-validation/3.0/apidocs/jakarta/validation/constraints/package-summary)


