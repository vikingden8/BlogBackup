---
title: The Difference Between “is” and “==” in Python
date: 2018-01-23 11:15:08
tags:
categories: "Python学习笔记"
---

Python has two operators for equality comparisons, “is” and “==” (equals). In this article I’m going to teach you the difference between the two and when to use each with a few simple examples.

When I was a kid, our neighbors had two twin cats.

Both cats looked seemingly identical—same charcoal fur, same piercing green eyes. Some personality quirks aside, you just couldn’t tell them apart just from looking at them. But of course they were two different cats, two separate beings, even though they looked exactly the same.

There’s a difference in meaning between equal and identical. And this difference is important when you want to understand how Python’s is and == comparison operators behave.

<!--more-->

**The == operator compares by checking for equality**: If these cats were Python objects and we’d compare them with the == operator, we’d get “both cats are equal” as an answer.

**The is operator, however, compares identities**: If we compared our cats with the is operator, we’d get “these are two different cats” as an answer.

But before I get all tangled up in this ball of twine of a cat analogy, let’s take a look at some real Python code.

First, we’ll create a new list object and name it a, and then define another variable  b that points to the same list object:

```Python
>>> a = [1, 2, 3]
>>> b = a
```

Let’s inspect these two variables. We can see they point to identical looking lists:

```Python
>>> a
[1, 2, 3]
>>> b
[1, 2, 3]
```

Because the two list objects look the same we’ll get the expected result when we compare them for equality using the == operator:

```Python
>>> a == b
True
```

However, that doesn’t tell us whether a and b are actually pointing to the same object. Of course, we know they do because we assigned them earlier, but suppose we didn’t know—how might we find out?

The answer is comparing both variables with the is operator. This confirms both variables are in fact pointing to one list object:

```Python
>>> a is b
True
```

Let’s see what happens when we create an identical copy of our list object. We can do that by calling list() on the existing list to create a copy we’ll name c:

```Python
>>> c = list(a)
```

Again you’ll see that the new list we just created looks identical to the list object pointed to by a and b:

```Python
>>> c
[1, 2, 3]
```

Now this is where it gets interesting—let’s compare our list copy c with the initial list a using the == operator. What answer do you expect to see?

```Python
>>> a == c
True
```

Okay, I hope this was what you expected. What this result tells us is that c and a have the same contents. They’re considered equal by Python. But are they actually pointing to the same object? Let’s find out with the is operator:

```Python
>>> a is c
False
```

Boom—this is where we get a different result. Python is telling us that c and a are pointing to two different objects, even though their contents might be the same.

So, to recap let’s try and break the difference between is and == down to two short definitions:

* An is expression evaluates to True if two variables point to the same (identical) object.

* An == expression evaluates to True if the objects referred to by the variables are equal (have the same contents).

Just remember, think of twin cats (dogs should work, too) whenever you need to decide between using is and == in Python. You’ll be fine.

From : [The Difference Between “is” and “==” in Python](https://dbader.org/blog/difference-between-is-and-equals-in-python)
