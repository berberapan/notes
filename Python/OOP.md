Object-Oriented Programming is a pattern for writing code. OOP and functional programming are all about making code easier to work with and understand. One of the most important things is to write the code in a way that's easy to understand for others.

```
Any fool can write code that a computer can understand. Good programmers write code that humans can understand.
 
 -- Martin Fowler
```

#### Classes

A class is a type but instead of being a built-in type they are a custom type that you define. The objects that are created from an class are instances of the class type.

You define a class with the keyword **class**. Class names starts with a capital letter.

```python
class Dog:
	legs = 4
	tail = 1

judith = Dog()
fido = Dog()

print(judith.legs) # Prints 4
print(fido.tail) # Prints 1
```

You can also have so called methods defined for a class. It's like a function but tied to the class and can only be used by the class.

```python
class Scoreboard:
	correct_answers = 0

	def correct_answer(self, good_answers)
		self.correct_answers += good_answers

new_scoreboard = Scoreboard()
new_scoreboard.correct_answer(5)
print(new_scoreboard.correct_answers) # prints 5
```

In the example above you can see that **self** is used. The first parameter of a method is always self, that's because it's the instance of the class the method is being called on. So self is a reference to the object. Methods doesn't need to return anything but they can. Often they're used to mutate properties.

Instead of defining properties like in the first example you usually define a constructor. To make a constructor in python you define a method with the name **__init__**. As a method can take parameters you can now configure the properties. 

```python
class Dog:
	def __init__(self, legs, tails):
		self.legs = legs
		self.tails = tails
```

When using the init method your variables are declared in the constructor so if you're changing values for an object you only update the instance. If you declare the variables directly in the class they operate more like global variables for the class so if you change a value it will change it for all instances.

#### Encapsulation

This is the practice of hiding complexity inside a black box so it becomes easier to focus on other things. The most basic example is the use of functions. Function caller only need to worry about arguments that needs to go in.

By default in Python all variables and methods in a class are public. To make something private you put **__** in front of the name. As Python is dynamic it doesn't enforce safeguards like other languages so it's more of a encapsulation by convention rather than by force. 

#### Abstraction

Abstraction is pretty similar to encapsulation. Abstraction handles complexity by hiding unnecessary details. It's about creating a simple interface for complex behaviour. So abstraction is more about reducing the complexity while encapsulation is more about maintaining the integrity of system internals.

#### Inheritance

Inheritance allows for a class to inherit properties and methods from another class. The class inheriting becomes the child of the other class which is a parent.

In the example below we can see that a dog is a child class of animal. All dogs are animals but not all animals are dogs.

```python
class Animal:
	def __init__(self, legs):
		self.legs = legs

class Dog(Animal):
	def __init__(self):
		super().__init__(4)
```

`super().__init__` can be used to reuse the parent's class constructor.

You should only use inheritance when you want the child class to inherit everything. If you only want it to share some of the functionality you should do something else.

A child class can't access a private property of it's parent class and needs to use a getter.

The parent class is not limited to one child but can have an infinite number of them. It's more common for inheritance trees to be wide than deep.

#### Polymorphism

Polymorphism is the ability of a variable, function or an object to take multiple forms. In the example below the methods have the same name in the same hierarchical tree but they act differently. So the children classes methods overrride the parent method.

```python
class Animal():
	def sound(self):
		print("the animal makes a sound")

class Dog(Animal):
	def sound(self):
		print("bark")

class Cat(Animal):
	def sound(self):
		print("meow")

for animal in [Animal(), Dog(), Cat()]:
	animal.sound()

# the animal makes a sound
# bark
# meow
```

Calling two methods with the same signature should look the same to the caller. For example if we do a parent class called shapes and got different shapes as children. If we want to have a method called area() we should just be able to call that for Rectangle, Triangle etc.

You can use operator overloading by using **__add__** method in a class.

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __add__(self, point):
        x = self.x + point.x
        y = self.y + point.y
        return Point(x, y)

p1 = Point(4, 5)
p2 = Point(2, 3)
p3 = p1 + p2
# p3 is (6, 8)
```

Here's a list of the translations for overloads

| Operation           | Operator | Method           |
| ------------------- | -------- | ---------------- |
| Addition            | +        | \_\_add\_\_      |
| Subtraction         | -        | \_\_sub\_\_      |
| Multiplication      | \*       | \_\_mul\_\_      |
| Power               | \*\*     | \_\_pow\_\_      |
| Division            | /        | \_\_truediv\_\_  |
| Floor Division      | //       | \_\_floordiv\_\_ |
| Modulo              | %        | \_\_mod\_\_      |
| Bitwise Left Shift  | <<       | \_\_lshift\_\_   |
| Bitwise Right Shift | >>       | \_\_rshift\_\_   |
| Bitwise AND         | &        | \_\_and\_\_      |
| Bitwise OR          | \|       | \_\_or\_\_       |
| Bitwise XOR         | ^        | \_\_xor\_\_      |
| Bitwise NOT         | ~        | \_\_invert\_\_   |
There is also a method for printing out a class instance. You use the method name **__str__** and in it return the values you want to show when the method is called. The method **__repr__** works in a similar way.

A few others, **__eq__** is for comparison and is like \=\=. \> is __gt__ and \< is **lt**.

You might see that most python programs got

```python
if __name__ == "__main__":
	main()
```

this ensures that the main function is only called when it's the file it's in that runs. It won't run as an imported module. This is the correct way to run main according to the python community.
 