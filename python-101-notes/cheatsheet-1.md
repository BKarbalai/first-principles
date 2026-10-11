# Cheatsheet 1: Python Fundamentals

Source: Codédex Python cheatsheet. Both cheatsheets are notes for the same course (Python 101).

## Print

```python
print('Hello World!')
print(1000)
print(3.14)
print(True)
```

## Comments

```python
# I'm a comment!

print('Gabby')  # I'm also one T.T
```

## Variables

```python
secret_num = 42          # int
gravity = 9.81           # float
username = '@snoopdogg'  # str
earth_is_flat = False    # bool
```

## Arithmetic Operations

```python
sum = 23 + 18
difference = 30 - 8
product = 10 * 2.5
quotient = 81 / 9
remainder = 76 % 4
exponent = 2 ** 3
```

## User Input

```python
username = input('Enter your name: ')
age = int(input('Enter your age: '))
```

## String Interpolation

```python
print(f'The square of {i} is {i*i}')
```

## Control Flow

```python
if grade >= 90:
  print('A')
elif grade >= 80:
  print('B')
elif grade >= 70:
  print('C')
else:
  print('D')
```

## Relational Operators

```python
a == b  # equal to
a != b  # not equal to
a > b   # greater than
a < b   # less than
a >= b  # greater than or equal to
a <= b  # less than or equal to
```

## Random Number

```python
import random

num = random.randint(1, 9)
```

## Logical Operators

```python
a and b  # True if both are true
a or b   # True if at least one is True
not a    # True if a is false
```

## Loops

```python
# While loop
while coffee < 1:
  print('Tired of Python')

# For loop
for i in range(10):
  print(i)
```

---
[Index](README.md) | [Next →](cheatsheet-2.md)
