---
title: XPath in Selenium WebDriver Complete Tutorial
date: 2017-08-10 22:49:06
tags:
categories: "测试开发"
---

In Selenium automation, if the elements are not found by the general locators like id, class, name, etc. then XPath is used to find an element on the web page .

In this tutorial, we will learn about the xpath and different XPath expression to find the complex or dynamic elements, whose attributes changes dynamically on refresh or any operations.

<!--more-->

### What is XPath

XPath is defined as XML path. It is a syntax or language for finding any element on the web page using XML path expression. XPath is used to find the location of any element on a webpage using HTML DOM structure. The basic format of XPath is explained below with screen shot.

![](/images/categories/test/010/032816_0758_XPathinSele1.png)

**Syntax for XPath**:

XPath contains the path of the element situated at the web page. Standard syntax for creating XPath is.

>Xpath=//tagname[@attribute='value']

* **//** : Select current node.

* **Tagname**: Tagname of the particular node.

* **@**: Select attribute.

* **Attribute**: Attribute name of the node.

* **Value**: Value of the attribute.

To find the element on web pages accurately there are different types of locators:

* **ID**: To find the element by ID of the element

* **Classname**: To find the element by Classname of the element

* **Name**: To find the element by name of the element

* **Link text**: To find the element by text of the link

* **XPath**: XPath required for finding the dynamic element and traverse between various elements of the web page

* **CSS path**: CSS path also locates elements having no name, class or ID.

### Types of X-path

There are two types of XPath:

* **Absolute XPath** .

* **Relative XPath** .

#### Absolute XPath :

It is the direct way to find the element, but the disadvantage of the absolute XPath is that if there are any changes made in the path of the element then that XPath gets failed.

The key characteristic of XPath is that it begins with the single forward slash(/) ,which means you can select the element from the root node.

Below is the example of an absolute xpath expression of the element shown in the below screen.

```sh
html/body/div[1]/section/div[1]/div/div/div/div[1]/div/div/div/div/div[3]/div[1]/div/h4[1]/b
```

![](/images/categories/test/010/032816_0758_XPathinSele2.png)

#### Relative xpath:

For Relative Xpath the path starts from the middle of the HTML DOM structure. It starts with the double forward slash (//), which means it can search the element anywhere at the webpage.

You can starts from the middle of the HTML DOM structure and no need to write long xpath.

Below is the example of a relative XPath expression of the same element shown in the below screen. This is the common format used to find element through a relative XPath.

```sh
//*[@class='featured-box']//*[text()='Testing']
```

![](/images/categories/test/010/032816_0758_XPathinSele3.png)

#### What are XPath axes.

XPath axes search different nodes in XML document from current context node. XPath Axes are the methods used to find dynamic elements, which otherwise not possible by normal XPath method having no ID , Classname, Name, etc.

Axes methods are used to find those elements, which dynamically change on refresh or any other operations. There are few axes methods commonly used in Selenium Webdriver like child, parent, ancestor, sibling, preceding, self, etc.

### Using XPath Handling complex & Dynamic elements in Selenium

#### Basic XPath:

XPath expression select nodes or list of nodes on the basis of attributes like ID , Name, Classname, etc. from the XML document as illustrated below.

> Xpath=//input[@name='uid']

![](/images/categories/test/010/032816_0758_XPathinSele4.png)

Some more basic xpath expressions:

```xml
Xpath=//input[@type='text']				
Xpath=//label[@id='message23']
Xpath=//input[@value='RESET']
Xpath=//*[@class='barone']
Xpath=//a[@href='http://demo.guru99.com/']
Xpath=//img[@src='//cdn.guru99.com/images/home/java.png']
```

#### Contains()

Contains() is a method used in XPath expression. It is used when the value of any attribute changes dynamically, for example, login information.

The contain feature has an ability to find the element with partial text as shown in below example.

In this example, we tried to identify the element by just using partial text value of the attribute. In the below XPath expression partial value 'sub' is used in place of submit button. It can be observed that the element is found successfully.

Complete value of 'Type' is 'submit' but using only partial value 'sub'.

```XML
Xpath=//*[contains(@type,'sub')]  
```

Complete value of 'name' is 'btnLogin' but using only partial value 'btn'.

```XML
Xpath=.//*[contains(@name,'btn')]
```

In the above expression, we have taken the 'name' as an attribute and 'btn' as an partial value as shown in the below screenshot. This will find 2 elements (LOGIN & RESET) as their 'name' attribute begins with 'btn'.

![](/images/categories/test/010/032816_0758_XPathinSele5.png)

Similarly, in the below expression, we have taken the 'id' as an attribute and 'message' as a partial value. This will find 2 elements ('User-ID must not be blank' & 'Password must not be blank') as its 'name' attribute begins with 'message'.

```XML
Xpath=//*[contains(@id,'message')]
```

![](/images/categories/test/010/032816_0758_XPathinSele6.png)

In the below expression, we have taken the "text" of the link as an attribute and 'here' as a partial value as shown in the below screenshot. This will find the link ('here') as it displays the text 'here'.

```XML
Xpath=//*[contains(text(),'here')]
Xpath=//*[contains(@href,'guru99.com')]
```

![](/images/categories/test/010/032816_0758_XPathinSele7.png)

#### Using OR & AND:

In OR expression, two conditions are used, whether 1st condition OR 2nd condition should be true. It is also applicable if any one condition is true or maybe both. Means any one condition should be true to find the element.

In the below XPath expression, it identifies the elements whose single or both conditions are true.

```XML
Xpath=//*[@type='submit' OR @name='btnReset']
```

Highlighting both elements as "LOGIN " element having attribute 'type' and "RESET" element having attribute 'name'.

![](/images/categories/test/010/032816_0758_XPathinSele8.png)

In AND expression, two conditions are used, both conditions should be true to find the element. It fails to find element if any one condition is false.

```XML
Xpath=//input[@type='submit' AND @name='btnLogin']
```

In below expression, highlighting 'LOGIN' element as it having both attribute 'type' and 'name'.

![](/images/categories/test/010/032816_0758_XPathinSele9.png)

#### Start-with function:

Start-with function finds the element whose attribute value changes on refresh or any operation on the webpage. In this expression, match the starting text of the attribute is used to find the element whose attribute changes dynamically. You can also find the element whose attribute value is static (not changes).

For example -: Suppose the ID of particular element changes dynamically like:

```XML
Id=" message12"

Id=" message345"

Id=" message8769"
```

and so on.. but the initial text is same. In this case, we use Start-with expression.

In the below expression, there are two elements with an id starting "message"(i.e., 'User-ID must not be blank' & 'Password must not be blank'). In below example, XPath finds those element whose 'ID' starting with 'message'.

```XML
Xpath=//label[starts-with(@id,'message')]
```

![](/images/categories/test/010/032816_0758_XPathinSele10.png)

#### Text():

In this expression, with text function, we find the element with exact text match as shown below. In our case, we find the element with text "UserID

```XML
Xpath=//td[text()='UserID']
```

![](/images/categories/test/010/032816_0758_XPathinSele11.png)

### Summary:

XPath is required to find an element on the web page as to do an operation on that particular element.

There are two types of XPath:

* Absolute XPath

* Relative XPath

XPath Axes are the methods used to find dynamic elements, which otherwise not possible to find by normal XPath method

XPath expression select nodes or list of nodes on the basis of attributes like ID , Name, Classname, etc. from the XML document .

From:[XPath in Selenium WebDriver: Complete Tutorial](https://www.guru99.com/xpath-selenium.html)
