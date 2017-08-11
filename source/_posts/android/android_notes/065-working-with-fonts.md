---
title: Working with Fonts
date: 2017-08-11 17:39:00
tags:
categories: "Android学习笔记"
---

**Android O introduces a new feature, Fonts in XML, which lets you use fonts as resources. Android O also provides a mechanism to retrieve information related to system fonts and provide file descriptors**.

### Fonts in XML

Android O lets you bundle fonts as resources by adding the font file in the **res/font/** folder. These fonts are compiled in your R file and are automatically available in Android Studio. You can access the font resources with the help of a new resource type, font. For example, to access a font resource, use **@font/myfont**, or **R.font.myfont**.

To add fonts as resources, perform the following steps in the Android Studio:

* Right-click the **res** folder and go to **New** > **Android resource directory**. The New Resource Directory window appears.

* In the Resource type list, select **font**, and then click **OK**.

>Note: The name of the resource directory must be font.

<!--more-->

![](/images/categories/android/android_notes/065/resource-directory-font.png)

* Add your font files in the **font** folder.

The folder structure below generates _R.font.dancing_script_, _R.font.lobster_, and _R.font.typo_graphica_.

![](/images/categories/android/android_notes/065/font-files-structure.png)

* Double-click a font file to preview the file's fonts in the editor.

![](/images/categories/android/android_notes/065/preview-font.png)

### Creating a font family

A font family is an XML file that contains multiple font files along with its style and weight details. You can access the font family as a single unit.

To create a font family, perform the following steps in the Android Studio:

* Right-click the **font** folder and go to **New** > **Font resource file**. The New Resource File window appears.

* Enter the file name, and then click **OK**. The new font resource XML opens in the editor.

* Enclose each font file, style, and weight attribute in the <font> element. The following XML illustrates adding font-related attributes in the font resource XML:

```XML
<?xml version="1.0" encoding="utf-8"?>
<font-family xmlns:android="http://schemas.android.com/apk/res/android">
    <font
        android:fontStyle="normal"
        android:fontWeight="400"
        android:font="@font/lobster_regular" />
    <font
        android:fontStyle="italic"
        android:fontWeight="400"
        android:font="@font/lobster_italic" />
</font-family>

```

### Using fonts in XML layouts

You can easily use your fonts, either a single font file or a font from a font family, in a TextView object or in styles. To attach fonts to the TextView or in styles, use the fontFamily attribute.

>Note: When you use a font family, the TextView switches on its own, as needed, to use the font files from that family.

#### Adding fonts to a TextView

To set a font for the TextView, do one of the following:

* Open the layout XML file, and then use the fontFamily attribute to access the font file.

```XML
<TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:fontFamily="@font/lobster"/>
```

* Open the Properties window to set the font for the TextView.

1.Select a view to open the Properties window.

>Note: The Properties window is available only when the design editor is open. Ensure you've selected the Design tab at the bottom of the window.

2.Expand the textAppearance property, and then select the font from the fontFamily list.

![](/images/categories/android/android_notes/065/property-window.png)

The Android Studio layout preview, shown in the rightmost pane of Figure 5, allows you to preview the font set in the TextView.

![](/images/categories/android/android_notes/065/xml-font-preview.png)

#### Adding fonts to style

Open the styles.xml, and use the fontFamily attribute to access the font file.

```xml
<style name="customfontstyle" parent="@android:style/TextAppearance.Small">
    <item name="android:fontFamily">@font/lobster</item>
</style>
```

#### Using fonts programmatically

You can retrieve fonts by using the getFont(int) method, where you need to pass the resource identifier of the font that you want to retrieve. This method returns a Typeface object. This encodes the first of the weight or style variants of your font, if it is a font family. You can then use the Typeface.create(typeface, style) method to retrieve specific styles.

>Note: The TextView already does this for you.

```java
Typeface typeface = getResources().getFont(R.font.myfont);
textView.setTypeface(typeface);
```

### Retrieving system fonts

The fonts.xml file describes all the fonts that are preinstalled on the device and their respective details such as family name, weight, and style. Until now, Android didn't expose public APIs to retrieve such system font data. So, some apps accessed it directly to find the system fonts and loaded them on their own instead of using the android font stack. Android O offers new APIs that expose the information in fonts.xml as well as provide FileDescriptor objects to the font files. Apps that directly load fonts are encouraged to move away from that practice and start using the font enumeration API. The new APIs are:

* **FontManager.getSystemFonts()**: Retrieves information related to system fonts and provides file descriptors.

* **FontConfig**: Data structure that contains all the information related to fonts.

### Accessing APIs from your app

To access APIs from your app, call the following methods:

```java
FontManager fontManager = context.getSystemService(FontManager.class);
FontConfig systemFontsData = fontManager.getSystemFonts();
```

>Note: The FontConfig class includes data on all the defined font families and, for each font within a family, the details about the font's weight and style. It also provides an open file descriptor to the font file. Callers of the API should ensure they close the given file descriptors once they have finished using them.

From:[https://developer.android.com/preview/features/working-with-fonts.html#retrieving-system-fonts](https://developer.android.com/preview/features/working-with-fonts.html#retrieving-system-fonts)
