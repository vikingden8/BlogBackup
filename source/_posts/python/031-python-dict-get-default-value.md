---
title: 使用Python字典中的get方法返回默认值
date: 2018-07-30 19:44:08
tags:
categories: "Python学习笔记"
---

假如有下面的数据结构来表示用户的ID对应的用户名：

```Python
name_for_userid = {
    382: "Alice",
    950: "Bob",
    590: "Dilbert",
}
```

现在编写一个 greeting() 函数根据传入的用户ID返回用户名。我们首先想到的实现方式是：

```Python
def greeting(userid):
    return "Hi %s!" % name_for_userid[userid]
```

<!--more-->

这个方法只适合于用户的ID在字典 name_for_userid 的keys中，若传入不存在的键值，则会抛出异常，如下：

```Python
>>> greeting(382)
"Hi Alice!"

>>> greeting(33333333)
KeyError: 33333333
```

我们来修改下 greeting() 的实现，当传入的键值不存在时，返回默认的问候语。我们首先想到的就是 “key in dict” 的成员检查实现方式：

```Python
def greeting(userid):
    if userid in name_for_userid:
        return "Hi %s!" % name_for_userid[userid]
    else:
        return "Hi, user not exist!"

>>> greeting(382)
"Hi Alice!"

>>> greeting(33333333)
"Hi, user not exist!"
```

上面的实现方式达到了我们想要的结果，但是并不好。

* 首先，它不高效，因为在执行过程中遍历了字典两次；

* 其次，看起来比较的繁琐，比如有两个 "Hi" ;

* 最后，这样的代码并不Pythonic，原因是Python官方建议使用 “easier to ask for forgiveness than permission” (EAFP) 的代码风格；

所以，尝试使用 EAFP 的方式来实现，如下：

```Python
def greeting(userid):
    try:
        return "Hi %s!" % name_for_userid[userid]
    except KeyError:
        return "Hi, user not exist!"
```

同样地，上面的实现方式应该也是可以的，但是，我们仍可以用一种简单的方式来实现。也就是，Python中的字典有一个 get() 方法，当在键不存在时，可以提供一个默认值返回。

```Python
def greeting(userid):
    return "Hi %s!" % name_for_userid.get(userid, ", user not exist!")
```

当在使用 get() 方法时，首先会去检查当前的键是否在字典中存在。若存在，则返回键所对应的值；若不存在，则返回 get() 方法中提供的默认值。

如你所见，greeting() 方法的执行效果跟之前的一样达到了需求：

```Python
>>> greeting(950)
"Hi Bob!"

>>> greeting(333333)
"Hi, user not exist!"
```

greeting() 最后的实现方式看起来更加的简洁，明了。

翻译自：[Using get() to return a default value from a Python dict](https://dbader.org/blog/python-dict-get-default-value)
