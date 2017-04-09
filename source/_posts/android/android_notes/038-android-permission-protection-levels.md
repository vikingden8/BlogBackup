---
title: Android Permission Protection Levels
date: 2017-04-09 14:55:49
tags:
categories: "Android学习笔记"
---

### Normal

**Value : 0**

**Description:**

A somewhat low-risk permission that gives an application access to isolated
application-level features, with minimal risk to other applications, the
system, or the user. The system automatically grants this type of permission
to a requesting application at installation, without asking for the user’s
explicit approval (though the user always has the option to review these
permissions before installing).

<!--more-->

### Dangerous

**Value : 1**

**Description:**

A higher risk permission that gives a requesting application access to
private user data or control over the device in a way that can negatively
impact the user. Because this type of permission introduces potential risk,
the system may not automatically grant it to the requesting application. Any
dangerous permissions requested by an application may be displayed to the
user and require confirmation before proceeding, or some other approach
may be taken so the user can avoid automatically allowing the use of such
facilities.

### Signature

**Value : 2**

**Description:**

The system will grant this permission only if the requesting application
is signed with the same certificate as the application that declared the
permission. If the certificates match, the system automatically grants
the permission without notifying the user or asking for the user’s explicit
approval.

### SignatureOrSystem

**Value : 3**

**Description:**

The system grants this permission only to packages in the Android system
image or that are signed with the same certificates. Please avoid using this
option because the signature protection level should be sufficient for most
needs, and it works regardless of exactly where applications are installed.
This permission is used for certain special situations where multiple vendors
have applications built into a system image, and these applications need to
share specific features explicitly because they are being built together

### References

<<Android App Security>>-Chapter 3:Android Security Architecture

[https://developer.android.com/guide/topics/manifest/permission-element.html](https://developer.android.com/guide/topics/manifest/permission-element.html)
