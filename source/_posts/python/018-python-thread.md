---
title: Python中的线程
date: 2018-03-29 19:25:08
tags:
categories: "Python学习笔记"
---

Python中的线程主要有两种实现方式。

### 继承threading.Thread

步骤：

* 创建一个threading.Thread的子类

* 覆盖__init__(self [,args])

* 重写run方法 

```python
viking@ubuntu:~/work/python/thread$ cat test_thread.py 
#!/usr/bin/python
# coding = utf-8

import threading
import time

class MyThread(threading.Thread):
    def __init__(self, threadId, name):
        threading.Thread.__init__(self)
        self.threadId = threadId
        self.name = name

    def run(self):
        print('starting...' + self.name)
        time.sleep(3)
        print('exiting...' + self.name)


t1 = MyThread(1, 'Thread-1')
t2 = MyThread(2, 'Thread-2')

t1.start()
t2.start()

t1.join()
t2.join()

print('Finish test threading...')
viking@ubuntu:~/work/python/thread$
```

<!--more-->

输出：

```
viking@ubuntu:~/work/python/thread$ python test_thread.py 
starting...Thread-1
starting...Thread-2
exiting...Thread-1
exiting...Thread-2
Finish test threading...
```

### 任务函数作为参数传递到Thread方法中

```python
viking@ubuntu:~/work/python/thread$ cat test_thread_2.py 
#!/usr/bin/python
# coding = utf-8

import threading
import time

def task(threadId, name):
    print('starting...' + name)
    time.sleep(3)
    print('exiting...' + name)

t1 = threading.Thread(target= task, args=(1, 'Thread-1'))
t2 = threading.Thread(target= task, args=(2, 'Thread-2'))

t1.start()
t2.start()

t1.join()
t2.join()

print('Thread testing all done!')

viking@ubuntu:~/work/python/thread$
```

输出：

```
viking@ubuntu:~/work/python/thread$ python test_thread_2.py 
starting...Thread-1
starting...Thread-2
exiting...Thread-1
exiting...Thread-2
Thread testing all done!
```





