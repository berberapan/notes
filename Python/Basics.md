##### Print

**print** keyword outputs text to terminal

```python
print("example")

# >> example
```

##### Variable

To set a variable you just set the variable name = the variable value. Python is dynamically typed and doesn't need you to set the data type.

```python
example_variable = "example"
example_number = 10
```

##### Comments

Comments are added by prefacing with '#' or '"""' for multi-line. Multi-line also needs to be closed with '""""'.

```python
# Example comment

"""
	Example
	More than
	1 line
"""
```

##### F-string

If you want a dynamic value in a string you can use an f-string.

```python
num = 10
print(f"The number is {num}")

# >> The number is 10
```

##### None

None is value that can be set to variables. It's not the same as a zero value but represents that there is no value.

```python
example = None
```

##### Functions

Functions are declared with **def** keyword. You don't need to set return value. Functions needs to be defined before called so entry point should be called last.

```python
def example_square(num):
	return num * num

def example_addition(x, y):
	return x + y
```
