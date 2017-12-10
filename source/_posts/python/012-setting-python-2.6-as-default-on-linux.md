---
title: Setting python 2.6 as default on Linux
date: 2017-04-15 22:37:08
tags:
categories: "Python学习笔记"
---

From : [Setting python 2.6 as default on Linux](http://web.mit.edu/6.00/www/handouts/pybuntu.html)

If you have a later version than 2.6 you'll need to set 2.6 as the default Python. Later versions would be 2.7 and 3.1; see what you have by typing
```
python -V
```

at the terminal. For purposes of this example we'll assume you have 3.1 installed. You'll next need to execute the following commands:

```
sudo apt-get install python2.6 idle-python2.6
sudo update-alternatives --install /usr/bin/python python /usr/bin/python3.1 1
sudo update-alternatives --install /usr/bin/python python /usr/bin/python2.6 10
sudo update-alternatives --config python
```
<!--more-->
This last command will allow you to choose which version of python to use by default. If you have done everything above correctly, python2.6 should already be set as the default. If it is not, choose it to be the default. From now on, runningpython should start version 2.6.

Undoing These Changes

In some cases (e.g., installing or updating certain packages), you'll get an error message if you've run the commands above. To update these packages, you'll have to temporarily undo these changes. Here's how to do that:

```
sudo update-alternatives --remove-all python
sudo ln -s python3.1 /usr/bin/python
```

Once you're done updating these packages, execute the commands at the top to set python2.6 as the default again.