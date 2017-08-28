---
title: Connecting to and Disconnecting from the Server
date: 2017-08-28 23:10:50
categories: "数据库"
---


To connect to the server, you will usually need to provide a MySQL user name when you invoke **mysql** and, most likely, a password. If the server runs on a machine other than the one where you log in, you will also need to specify a host name. Contact your administrator to find out what connection parameters you should use to connect (that is, what host, user name, and password to use). Once you know the proper parameters, you should be able to connect like this:

```sh
shell> mysql -h host -u user -p
Enter password: ********
```

<!--more-->

_host_ and _user_ represent the host name where your MySQL server is running and the user name of your MySQL account. Substitute appropriate values for your setup. The ******** represents your password; enter it when **mysql** displays the Enter password: prompt.

If that works, you should see some introductory information followed by a _mysql>_ prompt:

```sh
shell> mysql -h host -u user -p
Enter password: ********
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 25338 to server version: 5.7.20-standard

Type 'help;' or '\h' for help. Type '\c' to clear the buffer.

mysql>
```

The _mysql>_ prompt tells you that **mysql** is ready for you to enter SQL statements.

If you are logging in on the same machine that MySQL is running on, you can omit the host, and simply use the following:

```sh
shell> mysql -u user -p
```

If, when you attempt to log in, you get an error message such as **ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/tmp/mysql.sock'**, it means that the MySQL server daemon (Unix) or service (Windows) is not running. Consult the administrator or see the section of [Chapter 2, Installing and Upgrading MySQL](https://dev.mysql.com/doc/refman/5.7/en/installing.html) that is appropriate to your operating system.

For help with other problems often encountered when trying to log in, see [Section B.5.2, “Common Errors When Using MySQL Programs”](https://dev.mysql.com/doc/refman/5.7/en/common-errors.html).

Some MySQL installations permit users to connect as the anonymous (unnamed) user to the server running on the local host. If this is the case on your machine, you should be able to connect to that server by invoking mysql without any options:

```sh
shell> mysql
```

After you have connected successfully, you can disconnect any time by typing **QUIT** (or **\q**) at the _mysql>_ prompt:

```sh
mysql> QUIT
Bye
```

On Unix, you can also disconnect by pressing **Control+D**.
