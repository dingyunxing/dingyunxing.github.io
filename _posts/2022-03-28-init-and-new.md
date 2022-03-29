---
layout: post
title: Difference between __init__() and __new__() method in Python
category: Dev
tags: [python, OOP]
---

### Background
In Pyhton, there are some built-in functions when creating a class that looks like `__func_name__`. There are two of them, `__new__` and `__init__`. I found it's very interesting that we seldomly use `__new__` but mostly just use `__init__`. But actually they are bit different and are all useful for different senarios.
<!-- more -->

Most object-orented programming languages such as Java, C#, etc have the concept of a constructor - a method that creates and initialise the instance that created by the class. However, Python is different, it splits these into two different methods, the constructor is `__new__` while the initializer is `__init__`. But why we usually only use `__init__` but not `__new__`? Because the `__new__` is kind like hidden but actually already automatially excuted when we create a new instance. Let's start with a simple example:


#### Example 1:
```python
class A:
  def __init__(self):
    print("The init method has been excuted")

a = A()
```
**Output**:

`The init method has been excuted`

Ok, definately the init method has been excuted. Now let's add a `__new__` method that similar to `__init__`:

```python
class A:
  def __new__(self):
    print("The new method has been excuted")

  def __init__(self):
    print("The init method has been excuted")

a = A()
```
**Output**:

`The new method has been excuted`

Now we can see the `__new__` method kind like block the `__init__`. That's because what the `__new__` does is return a object(self) and pass to `__init__`. Also, we need to keep in mind that, `__new__` is a cls method that should pass `cls` instead of `self`. Although this is not that essential in Python but highly recommended to use the right word. 

Let's see what actually the `__new__` method does that inherit from base class or `Object` from Python 3. (Although we didn't say anything when define a class in Python 3, but `class A: = class A(object):`, or we can say all the class inherit from class object).
#### Example 2:
```python
class A:
  def __new__(cls):
    print("The new method has been excuted")
    return object.__new__(cls)

  def __init__(self):
    print("The init method has been excuted")

a = A()
```
**Output**:

`The new method has been excuted`

`The init method has been excuted`

Cool now we can see both methods have run. This is because a self instance was returned/created by `__new__` method and then passed to the `__init__`. This is how the Python actually create a object from a class. We usually don't write the `__new__` method but can directly initialize an object using `__init__` is because it will inherit the new method from base(Object).

Let's see another example that init a instance with params:

#### Example 3:
```python
class A:
  def __new__(cls, *args, **kwargs):
    return object.__new__(cls)

  def __init__(self, x, y, z):
      self.x = x
      self.y = y
      self.z = z
    

a = A(x=1, y=2, z=3)
print(a.x)
print(a.y)
print(a.z)
```
**Output**

`1`

`2`

`3`


### When to use `__new__`

Now the question is when shall we define a `__new__` method ourselves but not using the default one? Acutally it all depends. As we can see above, if we implement a `__new__` without any return value, it will not reture a self so that all the method below that used for instances will not be available anymore. 

Another very useful senario is when we just allow the class to create only one instance or `Singleton` in OO [Design Patterns](https://www.oodesign.com/)

#### Example 4:
Let's first see a class without `__new__`:
```python
class Singleton:
    ...

s1 = Singleton()
s2 = Singleton()
print(s1)
print(s2) 
```
**Output**

`<__main__.Singleton object at 0x7f86b0a93370>`

`<__main__.Singleton object at 0x7f86b0a935e0>`

We can see the two instance `s1` and `s2` are point to two different memory address.

Now we add a refined `__new__` method:

```python
class Singleton:
    _instance = None
    def __new__(cls, *args, **kwargs):
        if cls._instance is None:
            cls._instance = object.__new__(cls)

        return cls._instance

s1 = Singleton()
s2 = Singleton()
print(s1)
print(s2) 
```
**Output**

`<__main__.Singleton object at 0x7f870061d4c0>`

`<__main__.Singleton object at 0x7f870061d4c0>`

Now we see we can only create one instance from the class.
