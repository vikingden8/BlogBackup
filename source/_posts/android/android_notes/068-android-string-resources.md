---
title: Android String Resources-Plurals
date: 2017-12-10 21:33:36
tags:
categories: "Android学习笔记"
---


### Quantity Strings (Plurals)

Different languages have different rules for grammatical agreement with quantity. In English, for example, the quantity 1 is a special case. We write "1 book", but for any other quantity we'd write "n books". This distinction between singular and plural is very common, but other languages make finer distinctions. The full set supported by Android is zero, one, two, few, many, and other.

The rules for deciding which case to use for a given language and quantity can be very complex, so Android provides you with methods such as getQuantityString() to select the appropriate resource for you.

Although historically called "quantity strings" (and still called that in API), quantity strings should only be used for plurals. It would be a mistake to use quantity strings to implement something like Gmail's "Inbox" versus "Inbox (12)" when there are unread messages, for example. It might seem convenient to use quantity strings instead of an if statement, but it's important to note that some languages (such as Chinese) don't make these grammatical distinctions at all, so you'll always get the other string.

<!--more-->

The selection of which string to use is made solely based on grammatical necessity. In English, a string for zero is ignored even if the quantity is 0, because 0 isn't grammatically different from 2, or any other number except 1 ("zero books", "one book", "two books", and so on). Conversely, in Korean only the other string is ever used.

Don't be misled either by the fact that, say, two sounds like it could only apply to the quantity 2: a language may require that 2, 12, 102 (and so on) are all treated like one another but differently to other quantities. Rely on your translator to know what distinctions their language actually insists upon.

It's often possible to avoid quantity strings by using quantity-neutral formulations such as "Books: 1". This makes your life and your translators' lives easier, if it's an acceptable style for your application.

>Note: A plurals collection is a simple resource that is referenced using the value provided in the name attribute (not the name of the XML file). As such, you can combine plurals resources with other simple resources in the one XML file, under one <resources> element.

SYNTAX:

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <plurals
        name="plural_name">
        <item
            quantity=["zero" | "one" | "two" | "few" | "many" | "other"]
            >text_string</item>
    </plurals>
</resources>
```

* **zero**	When the language requires special treatment of the number 0 (as in Arabic).

* **one**	When the language requires special treatment of numbers like one (as with the number 1 in English and most other languages; in Russian, any number ending in 1 but not ending in 11 is in this class).

* **two**	When the language requires special treatment of numbers like two (as with 2 in Welsh, or 102 in Slovenian).

* **few**	When the language requires special treatment of "small" numbers (as with 2, 3, and 4 in Czech; or numbers ending 2, 3, or 4 but not 12, 13, or 14 in Polish).

* **many**	When the language requires special treatment of "large" numbers (as with numbers ending 11-99 in Maltese).

* **other**	When the language does not require special treatment of the given quantity (as with all numbers in Chinese, or 42 in English).

EXAMPLE:

```XML
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <plurals name="numberOfSongsAvailable">
        <!--
             As a developer, you should always supply "one" and "other"
             strings. Your translators will know which strings are actually
             needed for their language. Always include %d in "one" because
             translators will need to use %d for languages where "one"
             doesn't mean 1 (as explained above).
          -->
        <item quantity="one">%d song found.</item>
        <item quantity="other">%d songs found.</item>
    </plurals>
</resources>
```

Java code:

```java
int count = getNumberOfsongsAvailable();
Resources res = getResources();
String songsFound = res.getQuantityString(R.plurals.numberOfSongsAvailable, count, count);
```

When using the getQuantityString() method, you need to pass the count twice if your string includes string formatting with a number. For example, for the string %d songs found, the first count parameter selects the appropriate plural string and the second count parameter is inserted into the %d placeholder. If your plural strings do not include string formatting, you don't need to pass the third parameter to getQuantityString
