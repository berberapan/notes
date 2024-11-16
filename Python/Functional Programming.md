
Python is not a great language for functional programming for several reasons.

1. No [static typing](https://developer.mozilla.org/en-US/docs/Glossary/Static_typing).
2. Everything is [mutable](https://en.wikipedia.org/wiki/Immutable_object).
3. No [tail call optimization](https://exploringjs.com/es6/ch_tail-calls.html).
4. [Side effects](https://en.wikipedia.org/wiki/Side_effect_(computer_science)) are common.
5. Imperative and OOP styles abound in popular libraries.
6. [Purity](https://en.wikipedia.org/wiki/Pure_function) is not enforced (and sometimes not even encouraged).
7. [Sum Types](https://en.wikipedia.org/wiki/Algebraic_data_type) are hard to define.
8. [Pattern matching](https://en.wikipedia.org/wiki/Pattern_matching) is weak at best.

But as functional programming is a paradigm of useful techniques for writing better code a lot of things apply to languages that are not purely functional.

------

Functional programming is more about declaring what you want to happen rather than how you want it to happen. Imperative programming on the other hand declares both the what and the how.


**Example of imperative code:**

```python
car = create_car()
car.add_gas(10)
car.clean_windows()
```

**Example of functional code:**

```python
return clean_windows(add_gas(create_car()))
```

The difference in the examples above is that in the functional example the value of the car variable is never changed. Just new functions that returns new values with clean_windows returning the final result.

When using functional programming you strive to have your data as immutable. Once something is declared it shouldn't be able to be changed. Immutability usually means fewer bugs and more maintainable code.

If you're unsure if to use use functions or classes, default to functions. They serve different purposes. Go for classes if you need something long-lived and stateful. 

> **Classes** encourage you to think about the world as a hierarchical collection of objects. Objects bundle behaviour, data, and state together in a way that draws boundaries between instances of things, like chess pieces on a board.

> **Functions** encourage you to think about the world as a series of data transformations. Functions take data as input and return a transformed output. For example, a function might take the entire state of a chess board and a move as inputs, and return the new state of the board as output.

When debugging in functional programming it's advised to break up the code and use prints.

```python
def get_player_position(position, velocity, friction, gravity):
    return calc_gravity(calc_friction(calc_move(position, velocity), friction), gravity)
```

```python
def get_player_position(position, velocity, friction, gravity):
    moved = calc_move(position, velocity)
    slowed = calc_friction(moved, friction)
    final = calc_gravity(slowed, gravity)
    print("Given:")
    print(f"position: {position}, velocity: {velocity}, friction: {friction}, gravity: {gravity}")
    print("Results:")
    print(f"moved: {moved}, slowed: {slowed}, final: {final}")
    return final
```

#### First Class Functions

In Python, functions are just values. Therefore they can be assigned to a variable.

Anonymous functions in Python are called lambdas. They have no name and returns automatically. These are good to assign to a variable. In the example below the lambda takes the argument x and returns it's value plus one.

```python
add_one = lambda x: x + 1
print(add_one(2))
# 3
```

A programming language supports **first-class functions** when functions are treated like any other variable. That means functions can be passed as arguments to other functions, can be returned by other functions, and can be assigned to variables.

- **First-class function:** A function that is treated like any other value
- **Higher-order function:** A function that accepts another function as an argument or returns a function

**Map**

Map, filer and reduce are three commonly used high-order functions in functional programming. In Python the map function takes a function and an iterable as inputs and returns an iterator that applies the function to every item.

```python
def square(x):
    return x * x

nums = [1, 2, 3, 4, 5]
squared_nums = map(square, nums)
print(list(squared_nums))
# [1, 4, 9, 16, 25]
```

**Filter**

Filter takes a function and an iterable and returns a new iterable that only contains elements from the original iterable where the result from the function returned True.

```python
def is_even(x):
    return x % 2 == 0

numbers = [1, 2, 3, 4, 5, 6]
evens = list(filter(is_even, numbers))
print(evens)
# [2, 4, 6]
```

**Reduce**

You have to import the functools package to use the reduce. It takes a function and a list of values, it then applies the function to each value in the list, accumulating a single result as it goes.

```python
# import functools from the standard library
import functools

def add(sum_so_far, x):
    print(f"sum_so_far: {sum_so_far}, x: {x}")
    return sum_so_far + x

numbers = [1, 2, 3, 4]
sum = functools.reduce(add, numbers)
# sum_so_far: 1, x: 2
# sum_so_far: 3, x: 3
# sum_so_far: 6, x: 4
# 10 doesn't print, it's just the final result
print(sum)
# 10
```

High-order functions allows to avoid stateful iteration and mutations of variables. It's just a combining of functions to get the results. In imperative code you'd have to loop through the iterables that's inherently stateful.

**Intersect**

Intersect can be used with the intersection method in Python. It calculates the intersection of two sets.

```python
a = {1, 2, 3, 4}
b = {3, 4, 5, 6}
c = a.intersection(b)
print(c)
# {3, 4}
```

**Zip**

It takes two iterables and returns a new iterable where each element is a tuple containing one element from each of the original iterables.

```python
a = [1, 2, 3]
b = [4, 5, 6]

c = list(zip(a, b))
print(c)
# [(1, 4), (2, 5), (3, 6)]
```

#### Pure Functions

Pure functions always return the same value given the same arguments and running them causes no side effects. Ergo they don't do anything with anything that exists outside their scope.

Side effects are needed for a program to do anything for a user. But a good rule of thumb is to keep those side effects limited to a few places, near top level and have as many pure functions as possible.

So pure functions:

- Return the same result if given the same arguments. They are [deterministic](https://en.wikipedia.org/wiki/Deterministic_system).
- Do not change the external state of the program. For example, they do not change any variables outside of their scope.
- Do not perform any [I/O operations](https://en.wikipedia.org/wiki/Input/output) (like reading from disk, accessing the internet, or writing from the console).

These properties result in pure functions being easier to test, debug, and think about.

When passing a value into a function as an argument it can either be passed by reference (the function has access to  original value and can change it) or it's passed by value (the function only got access to a copy and changes won't affect the original)

These types are passed by **reference**:

- Lists
- Dictionaries
- Sets

These types are passed by **value**:

- Integers
- Floats
- Strings
- Booleans
- Tuples

Most collection types are passed by reference (except for tuples) and most primitive types are passed by value.

Pure functions shouldn't have any side effects so they shouldn't modify anything outside its scope. So when working with the types that are passed by reference you should return copies instead of changing the input value.

A **no-op** is a function that does nothing. If a function doesn't return anything it's probably impure and it's only reason to exist is to perform a side effect.

**Memoization** is basically a type of caching. The slower and more complex a function is, the more memoization can help speed things up. So for example if you have a function that adds two parameters, the 3 and 4 inputs will always result in 7. Therefore you could store it and as pure functions should always result in the same result given the same arguments it shouldn't be any issues. This is called referentially transparent, and all functions that are of that type can safely be memoized. But memoization is a tradeoff between memory and speed so it should only be used for slow and complex things as it makes the code harder to read.

#### Recursion

A recursive function is just a function that calls itself. In functional programming you iterate over an iterable with recursions. Functional programming avoids loops as they're stateful.

```python
def sum(nums):
    if len(nums) == 0:
        return 0
    return nums[0] + sum(nums[1:])

print(sum([1, 2, 3, 4, 5]))
# 15
```

Recursion is very useful when traversing tree-like items. Depth can be arbitrary, so looping over such things can be hard. File systems, nested dictionaries etc are good to use recursion on.

Some things to look out for when using recursion is **stack overflow**. Each call requires a bit of memory so if you recurse to deeply you can run out of stack in the memory which will crash the program. **Base case**, try to set a solid one when creating your recursion, otherwise you may end up with an infinite loop. **Slowness**, in many languages using recursions are slower than using loops because of the function calls requiring some memory. Some language like Kotlin got support for tail call but Python doesn't have support for it.

#### Function Transformations

A function transformation is when a function takes a function or many functions as input and returns a new function.

```python
def multiply(x, y):
    return x * y

def add(x, y):
    return x + y

# self_math is a higher order function
# input: a function that takes two arguments and returns a value
# output: a new function that takes one argument and returns a value
def self_math(math_func):
    def inner_func(x):
        return math_func(x, x)
    return inner_func

square_func = self_math(multiply)
double_func = self_math(add)

print(square_func(5))
# prints 25

print(double_func(5))
# prints 10
```

When should you use this? Only when it makes the code simpler than it would otherwise be. For example when having a function that gives out different patterns based on a formatter. It can make it easier to share common functionality.

It's also useful when creating a closure. 

#### Closures

A closure is a function that references variables from outside its own function body. A closure is just a function that keeps track of some values from the place it was defined, no matter where it was called.

```python
def concatter():
	doc = ""
	def doc_builder(word):
		# "nonlocal" tells Python to use the 'doc'
		# variable from the enclosing scope
		nonlocal doc
		doc += word + " "
		return doc
	return doc_builder

# save the returned 'doc_builder' function
# to the new function 'harry_potter_aggregator'
concat_string = concatter()
concat_string("Hello,")
concat_string("nice to")
concat_string("meet you")


print(concat_string("!"))
# Hello, nice to meet you!
```

The **nonlocal** keyword in the example is required to modify the variable from an enclosing scope in python. Most other languages doesn't need a keyword like that. Nonlocal is only used if you're reassigning a variable, so when you're dealing with an immutable value like a strin and integer. It's not used when you can modify the value like for a list or a set.

Closures are stateful so they're in many cases not pure functions as they can mutate state outside of their scope and have side effects.

#### Currying

Function currying is a kind of function transformation where we translate a single function that accepts multiple arguments into multiple functions that each accept a single argument.

```python
def sum(a, b):
  return a + b

print(sum(1, 2))
# prints 3
```

Same as above but curried

```python
def sum(a):
  def inner_sum(b):
    return a + b
  return inner_sum

print(sum(1)(2))
# prints 3
```

It looks and is a bit more complicated than the example above but it's used often to change a function's signature to make it conform to a shape.