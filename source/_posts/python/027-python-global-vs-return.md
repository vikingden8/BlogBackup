---
title: Python Global vs Return
date: 2018-04-08 21:41:08
tags:
categories: "Python学习笔记"
---

翻译自：[Global & Return](http://book.pythontips.com/en/latest/global_&_return.html)

You might have encountered some functions written in python which have a return keyword in the end of the function. Do you know what it does? It is similar to return in other languages. Lets examine this little function:

你或许已经在Python中碰到过有些函数在末尾会有return关键字。你知道它有什么用途吗？其实跟其他语言类似。来看一下下面的示例：

```python
def add(value1, value2):
    return value1 + value2

result = add(3, 5)
print(result)
# Output: 8
```

The function above takes two values as input and then output their addition. We could have also done:

上面的函数有两个输入参数，用途是返回两者之和。我们也可以这样去实现：

<!--more-->

```python
def add(value1,value2):
    global result
    result = value1 + value2

add(3,5)
print(result)
# Output: 8
```

So first lets talk about the first bit of code which involves the return keyword. What that function is doing is that it is assigning the value to the variable which is calling that function which in our case is result. In most cases and you won’t need to use the global keyword. However lets examine the other bit of code as well which includes the global keyword. So what that function is doing is that it is making a global variable result. What does global mean here? Global variable means that we can access that variable outside the scope of the function as well. Let me demonstrate it with an example:

先来看一下有return关键字的函数。有return函数的用途是把返回的值赋值给result。大多数情况下，我们都不需要使用到global关键字。我们来看一下上面出现了global关键字的函数。上面的函数主要是定义了一个全局的result变量，然后把两个参数的值相加赋值给result。那这里的global意味着什么呢？其实，global的用途就是我们可以在函数之外的作用域去访问它。下面用一个实例来演示：

```python
# first without the global variable
# 这里没有global关键字
def add(value1, value2):
    result = value1 + value2

add(2, 4)
print(result)

# Oh crap, we encountered an exception. Why is it so?
# the python interpreter is telling us that we do not
# have any variable with the name of result. It is so
# because the result variable is only accessible inside
# the function in which it is created if it is not global.

# 当我们尝试去打印出result的值时，会出现异常，会提示result没有定义
# 导致如此的原因就是如果没有使用global修饰的话，result变量的作用域只在函数的内部
Traceback (most recent call last):
  File "", line 1, in
    result
NameError: name 'result' is not defined

# Now lets run the same code but after making the result
# variable global
# 当我们使用global关键字修饰后，可以正常访问，无异常
def add(value1, value2):
    global result
    result = value1 + value2

add(2, 4)
result
6
```

So hopefully there are no errors in the second run as expected. In practical programming you should try to stay away from global keyword as it only makes life difficult by introducing unwanted variables to the global scope.

第二种情况不会有异常输出。在实际的程序开发中你应当避免使用global关键字。

* Multiple return values 多个返回值

So what if you want to return two variables from a function instead of one? There are a couple of approaches which new programmers take. The most famous approach is to use global keyword. Let’s take a look at a useless example:

如果需要在函数中返回的变量是多个，该怎么办呢？有多种方式来实现，也可以用丑陋的global关键字来实现，看下代码：

```python
def profile():
    global name
    global age
    name = "Danny"
    age = 30

profile()
print(name)
# Output: Danny

print(age)
# Output: 30
```

**Note:** Don’t try to use the above mentioned method. I repeat, don’t try to use the above mentioned method!

**注意:** 重申下，实际开发中不要使用上面类似的代码实现。

Some try to solve this problem by returning a tuple, list or dict with the required values after the function terminates. It is one way to do it and works like a charm:

解决这个问题的方法，可以返回元组，列表和字典，比如：

```python
def profile():
    name = "Danny"
    age = 30
    return (name, age)

profile_data = profile()
print(profile_data[0])
# Output: Danny

print(profile_data[1])
# Output: 30
```

Or by more common convention:

或者，更通用的写法：

```python
def profile():
    name = "Danny"
    age = 30
    return name, age

profile_name, profile_age = profile()
print(profile_name)
# Output: Danny
print(profile_age)
# Output: 30
```

This is a better way to do it along with returning lists and dicts. Don’t use global keyword unless you know what you are doing. global might be a better option in a few cases but is not in most of them.

除非你明确需要使用global关键字，否则你最好别使用它。