---
title: What is DDL, DML, DCL, DQL in Database?
date: 2017-08-28 22:58:00
categories: "数据库"
---

### **DDL (Data Definition Language)**

It is a set of SQL commands used to create, modify and delete database structure but not data. These are normally not used by a general user, who should be accessing the database via an application. These are normally used by DBA. These statements are immediate i.e they are not susceptible to ROLLBACK commands.

**Example**: CREATE, ALTER, DROP, TRUNCATE, COMMENT.

### **DML (Data Manipulation Language)**

These are SQL Statements that allows changing data within the database.

**Example**: INSERT, UPDATE, DELETE, CALL, EXPLAIN PLAN, LOCK

### **DCL (Data Controlling Language)**

It is the component of SQL statement that control access to the data and to the database.

**Example**: COMMIT, SAVEPOINT, ROLLBACK, SET TRANSACTION, GRANT/REVOKE

### **DQL (Data Query Language)**

It is the component of SQL statement that allows getting data from the database.

**Example**: SELECT
