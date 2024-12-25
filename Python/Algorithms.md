Algorithms in computer science are a finite sequence of well-defined, computer-implementable instructions. You can say an algorithm can be summarised like this:

- **Defined** - there is a specific sequence of steps that performs a task
- **Unambiguous** - there is a correct and incorrect interpretation of the steps
- **Implementable** - it can be executed using software and hardware 

When it comes to algorithms it's important to know how exponents works to understand the speed of execution.
x ^ 2 will keep growing faster and faster, while a 2 \* x growth is linear. Try to write code that doesn't grow quadratic as it sometimes leads to slowness (more CPU usage), sometimes more memory usage or sometimes a large storage requirement. Unless you work with cryptography or security were complexity waste resources of the attacker.

Big O is a way to compare algorithms. It describes the theoretical growth rate of the algorithm. Because it's supposed to be a general overlook you don't go into the details like constants when deciding the Big O.

![[bigo.png]]

- `O(1)` - constant
- `O(n)` - linear
- `O(n^2)` - squared
- `O(2^n)` - exponential
- `O(n!)` - factorial

Polynomial time algorithms can be useful (it's polynomial time if its runtime doesn't grow faster than n^k where k is any constant and n is the size of the input), but exponential time is almost always too slow to be used.

Polynomial Time is shortened to P. You can say that problems that fall into P are practical to solve on computers while those that don't are impractical as they will be slow and hard.

| Big-O          | Name           | Description                                                                                                                                                                                      |
| -------------- | -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **O(1)**       | constant       | **Best** The algorithm always takes the same amount of time, regardless of how much data there is. Example: Looking up an item in a list by index                                                |
| **O(log n)**   | logarithmic    | **Great** Algorithms that remove a percentage of the total steps with each iteration. Very fast, even with large amounts of data. Example: Binary search                                         |
| **O(n)**       | linear         | **Good** 100 items, 100 units of work. 200 items, 200 units of work. This is usually the case for a single, non-nested loop. Example: unsorted array search.                                     |
| **O(n log n)** | "linearithmic" | **Okay** This is slightly worse than linear, but not too bad. Example: mergesort and other "fast" sorting algorithms.                                                                            |
| **O(n^2)**     | quadratic      | **Slow** The amount of work is the square of the input size. 10 inputs, 100 units of work. 100 Inputs, 10,000 units of work. Example: A nested for loop to find all the ordered pairs in a list. |
| **O(n^3)**     | cubic          | **Slower** If you have 100 items, this does 100^3 = 1,000,000 units of work. Example: A triple nested for loop to find all the ordered triples in a list.                                        |
| **O(2^n)**     | exponential    | **Horrible** We want to avoid this kind of algorithm at all costs. Adding one to the input _doubles_ the amount of steps. Example: Brute-force guessing results of a sequence of `n` coin flips. |
| **O(n!)**      | factorial      | **Even More Horrible** The algorithm becomes so slow so fast, that is practically unusable. Example: Generating all the permutations of a list                                                   |


## Data Structures

#### Lists

- Lists are really good to use if you *append* and *grab index values* from them. Both of those cases are done **O(1)**. 
- When removing a value that's not the last index in a list the whole list has to shift which makes it **O(n)**.
- Searching for a value is also done in **O(n)** as it has to iterate over the full list.

#### Stacks

Stores ordered items. It only allows for an item to be added or removed from the top of the stack. You can refer to it as **LIFO** (Last In First Out).

A stack is like a list with fewer features. But all the features are **O(1)**.

|Operation|Big O|Description|
|---|---|---|
|push|O(1)|Add an item to the top of the stack|
|pop|O(1)|Remove and return the top item from the stack|
|peek|O(1)|Return the top item from the stack without modifying the stack|
|size|O(1)|Return the number of items in the stack|
Stack operations are limited: no searching, no sorting, no random access.


#### Queues

A queue only allows items to be added at the tail and removed at the head. Unlike stacks, queues uses **FIFO** (First In First Out). You can use the words enqueue and dequeue instead of push and pop, even though those works fine as well.

Just like a stack an optimised queue is **O(1)** for its operations.

| Operation | Big O | Description                                                      |
| --------- | ----- | ---------------------------------------------------------------- |
| push      | O(1)  | Add an item to the back of the queue                             |
| pop       | O(1)  | Remove and return the front item from the queue                  |
| peek      | O(1)  | Return the front item from the queue without modifying the queue |
| size      | O(1)  | Return the number of items in the queue                          |

#### Linked Lists

Unlike a list, a linked list's values are not next to each other in memory but instead got a reference to the next node. If a queue want to have O(1) for the push it needs to use a linked list. A linked list only need to change the reference for its head and tail and not shift the full list.

With that said linked list are usually a worse choice than a list if it's not a queue. They got more memory overhead, they are more complex to use and they have no random access.