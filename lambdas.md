# Python Functional Programming Concepts

## Lambda Functions

Lambda functions (also called anonymous functions) are small, one-line functions that don't require a formal `def` definition.

```python
# Syntax: lambda arguments: expression
square = lambda x: x * x
print(square(5))  # Output: 25
```

Key characteristics:
- Can take any number of arguments but only one expression
- Often used for short, simple operations
- Commonly passed as arguments to higher-order functions

## List Comprehensions

List comprehensions provide a concise way to create lists.

```python
# Basic syntax: [expression for item in iterable]
squares = [x**2 for x in range(10)]
# [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

# With condition
even_squares = [x**2 for x in range(10) if x % 2 == 0]
# [0, 4, 16, 36, 64]
```

Variations:
- Dictionary comprehensions: `{key: value for item in iterable}`
- Set comprehensions: `{expression for item in iterable}`

## Filter

The `filter()` function constructs an iterator from elements of an iterable for which a function returns true.

```python
# Syntax: filter(function, iterable)
numbers = [1, 2, 3, 4, 5, 6]
even_numbers = list(filter(lambda x: x % 2 == 0, numbers))
# [2, 4, 6]
```

## Map

The `map()` function applies a given function to each item of an iterable and returns an iterator.

```python
# Syntax: map(function, iterable)
numbers = [1, 2, 3, 4]
squares = list(map(lambda x: x**2, numbers))
# [1, 4, 9, 16]
```

## Reduce

The `reduce()` function (from `functools`) applies a function of two arguments cumulatively to the items of an iterable, from left to right, to reduce the iterable to a single value.

```python
from functools import reduce

# Syntax: reduce(function, iterable[, initializer])
numbers = [1, 2, 3, 4]
product = reduce(lambda x, y: x * y, numbers)
# 24 (1*2*3*4)
```

## Comparison Table

| Concept           | Purpose                                      | Returns                     |
|-------------------|----------------------------------------------|-----------------------------|
| Lambda            | Create small anonymous functions             | Function object             |
| List Comprehension | Create lists concisely                       | List                        |
| Filter            | Select items based on condition              | Iterator (filter object)    |
| Map               | Apply function to all items                  | Iterator (map object)       |
| Reduce            | Cumulatively apply function to reduce items  | Single value                |

These functional programming tools can make your code more concise and expressive when used appropriately, though they can also reduce readability if overused or used in complex ways.


# Multiple If Conditions in List Comprehensions

List comprehensions can include multiple conditional expressions to filter elements more precisely. There are two main ways to implement multiple conditions:

## 1. Multiple If Conditions (AND logic)

When you want **all** conditions to be true (logical AND), you can chain multiple `if` clauses:

```python
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# Numbers divisible by 2 AND greater than 5
result = [x for x in numbers if x % 2 == 0 if x > 5]
# Equivalent to: [x for x in numbers if x % 2 == 0 and x > 5]
print(result)  # Output: [6, 8, 10]
```

## 2. Combined Conditions with Logical Operators

You can also combine conditions using `and`, `or`, and `not`:

```python
# Numbers between 3 and 8 OR divisible by 5
result = [x for x in numbers if (3 < x < 8) or (x % 5 == 0)]
print(result)  # Output: [4, 5, 6, 7, 10]
```

## 3. If-Else Conditions in the Expression

When you need to transform values differently based on conditions (not just filter), put the conditional in the expression part:

```python
# Square even numbers, cube odd numbers
result = [x**2 if x % 2 == 0 else x**3 for x in numbers]
print(result)  # Output: [1, 4, 27, 16, 125, 36, 343, 64, 729, 100]
```

## Complex Example

```python
words = ['apple', 'banana', 'cherry', 'date', 'elderberry']

# Words that start with a vowel AND have length > 4 OR contain 'rr'
result = [word.upper() 
          for word in words 
          if (word[0] in 'aeiou' and len(word) > 4) 
          or 'rr' in word]
print(result)  # Output: ['APPLE', 'ELDERBERRY', 'CHERRY']
```

## Nested Conditions

You can nest conditions for more complex logic:

```python
numbers = [10, 20, 30, 40, 50, 60, 70]

# Numbers divisible by 20 OR (divisible by 10 and less than 35)
result = [x for x in numbers 
          if x % 20 == 0 
          or (x % 10 == 0 and x < 35)]
print(result)  # Output: [10, 20, 30]
```

Remember that while list comprehensions with multiple conditions are powerful, they can become hard to read if too complex. In such cases, consider using a regular `for` loop for better readability.
