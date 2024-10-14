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

##### Multiple returns

Python supports multiple returns in functions.

```python
def multi_return(x):
	y = x + 1
	z = x + 2
	return y, z
```

##### Operations

You can use basic math without any libraries.

```python
5 + 5
# 10

4 - 6
# -2

2 * 3
# 6

10 / 2
# 5.0  <- Division gives a float

23 // 5
# 4 <- Floor division

3 ** 2 
# 9 <- Exponents

7 % 2
# 1 <- Modulo
```

Python has got in-place operators to add, subtract, divide and multiply from a variable value.

```python
example = 1
example += 3
# example is now 4

example = 4
example /= 2
# example is now 2.0
```

##### Binary

Binary is prefixed with **0b**.

```python
five = 0b0101
```

Bitwise works like logical operators but applies to all bits in a value by column. It got & and | as operators. So in the example below with & it checks for each column. If both got 1 the bitwise will have 1 in that column, if both 0 it becomes 0, and if it's 1 and 0 the value for that column will be 0. The | operator does things in reverse. 1 - 1 equals a 1, 0 - 0 equals a 0 and 1 - 0 equals a 1.

```python
five = 0b0101
seven = 0b0111
five & seven
# equals 5
five | seven
# equals 7
```

##### Big numbers

You can use scientific notation in python. Add e followed by a positive or negative integer.

```python
print(2e4)
# prints 20000.0
```

You can also use underscores as delimiter when handling big numbers instead of commas that would result in a syntax error.

```python
num = 5_000_000
# if printed would show 5000000
```

##### Comparison

You compare values with operators like below.

```python
5 < 6 # evaluates to True
5 > 6 # evaluates to False
5 >= 6 # evaluates to False
5 <= 6 # evaluates to True
5 == 6 # evaluates to False
5 != 6 # evaluates to True
```

##### Logical operators

Use the keywords **and** and **or** to check so different inputs matches the logic.

```
True and True == True
True and False == False
False and False == False

True or True == True
True or False == True
False or False == False
```

##### If statement

These are used to check if certain conditions are met and do something in that case and if not do something else.

```python
if player1_score > player2_score:
	return "player1 wins"
return "player2 wins"
```

You can add **elif** (else if) to add more conditions. And finish it off with **else**. Else can usually be omitted though as that will be the condition that's left.

```python
if DIF_score > AIK_score:
	return "win"
elif DIF_score < AIK_score:
	return "lose"
else:
	return "draw"
```

##### For loop

You can loop code for a set condition. These are called for loops. 

```python
for i in range(1, 6):
	print(i)

# 1
# 2
# 3
# 4
# 5
```

Range also got an optimal parameter where you can set the step. If you set step a two for example it will step over every other iteration.

```python
for i in range(1, 5, 2):
	print(i)

# 1
# 3
# 5
```

When you don't need the index you can use an no-index or no-range syntax.

```python
for example in examples:
	print(example)

# example1
# example2
# example3
```
##### While loop

This loop continues until conditions are met. Be careful with the conditions so you're not creating a forever loop.

```python
while True:
print("x")

# x
# x
# x Goes on like this forever

x = 0
while x < 2:
	print(x)
	x += 1

# 0
# 1
# 2
```

##### List

A list is a collection of data. They are declared by using square brackets and items within are separated by commas. Lists can contain any type of data.

Indexing in Python starts with 0.

How long a list is can be checked with **len**.

You can add to an existing list with **append**.

You can remove the last index or specific one with **pop**. Pop also returns the value.

You can concatenate lists by just adding the together with a **+**.

To check if a value is present in a list you can use the keyword **in**.

You can delete items in the list with the **del** keyword. You can also use list slicing to remove more values.

```python
my_list = ["a", "b", "c"]

my_numbers = [
	1,
	2,
	3
]

my_numbers[1]
# 2

len(my_list)
# 3

my_list.append("d")
len(my_list)
# 4

example_pop = my_list.pop()
len(my_list)
# 3
# example_pop got value "d"

example_pop = my_list.pop(1)
# example_pop got value "b"

list_a = ["a", "b"]
list_b = ["b", "a"]
new_list = list_a + list_b
# ["a", "b", "b", "a"]

example_list = [1, 2 ,3]
print(2 in example_list)
# prints True

delete_list = [1, 2, 3, 4]
del delete[2]
# [1, 2, 4]
```

##### Min and Max

To check for a min or max value it's usually a good thing to assign a variable with the biggest or smallest value possible to compare. You can use **float("-inf")** and **float("inf")** for this.

##### List slice

You can slice lists with slicing operators.

```python
my_list[ start : stop : step ]

scores = [50, 70, 30, 20, 90, 10, 50]
print(scores[1:5:2])
# Prints [70, 20]
```

##### Tuples

Tuples are like lists a collection of data, values are unchangeable though. You create a tuple with round brackets. Tuples are often stored in small groups.

```python
my_tuple = (2, 20, 200)
```

##### Dictionaries

A dictionary in python is used to store key value pairs. It's initiated with curly brackets. Every key should be unique. If a value is repeated the second value just overwrites the first value.

To access a value you can write the key in square brackets. You can also add and delete values that way.

Dictionaries are ordered since Python 3.7.

```python
my_dict = {
	"name": "Joe",
	"number": 5
}

print(my_dict["name"])
# Joe
```

##### Sets

Sets are like lists but unordered and can only have unique items. Just like dictionaries they are initiated with curly brackets unless empty when you use **set**.

```python
example = {1, 2, 3}

empty_set = set()

example.add(4)

example.remove(4)
```

##### Try Catch

You can use try-catch blocks in python, if an exception is raised while in the block the program won't quit but execute the catch block.

You can also raise your own exceptions, but you shouldn't be doing raises and catches of those raises in the same block.

You can have more than one exception. The more specific ones should be ahead of the more general ones.

```python
try:
	1 / 0
except Exception as e:
	print(e)


raise Exception("my own error")
```

