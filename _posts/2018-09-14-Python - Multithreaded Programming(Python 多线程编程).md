---
layout:     post
title:      Python - 多线程编程 Multithreaded Programming
subtitle:   Python - 多线程编程
date:       2019-09-14
author:     LZB
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - Python
---

# 多线程编程 Multithreaded Programming #

运行多个线程类似于同时运行多个不同的程序，但具有以下好处 - 

- 进程中的多个线程与主线程共享相同的数据空间，因此可以比它们是单独的进程更容易地共享信息或相互通信。

- 线程有时称为轻量级进程，它们不需要太多内存开销; 它们比流程便宜。

线程有一个开头，一个执行序列和一个结论。 它有一个指令指针，可以跟踪当前运行的上下文。

- 它可以被抢占（中断）

- 当其他线程正在运行时，它可以暂时被搁置（也称为休眠） - 这称为让步。

## 开始一个新线程 ##
要生成另一个线程，您需要调用线程模块中可用的以下方法 -


    thread.start_new_thread ( function, args[, kwargs] )


此方法调用可以快速有效地在Linux和Windows中创建新线程。

方法调用立即返回，子线程启动并使用传递的args列表调用函数。 当函数返回时，线程终止。

在这里，args是一个参数元组; 使用空元组来调用函数而不传递任何参数。 kwargs是关键字参数的可选字典。

### 示例 ###

	#!/usr/bin/python
	
	import thread
	import time
	
	# Define a function for the thread
	def print_time( threadName, delay):
	   count = 0
	   while count < 5:
		  time.sleep(delay)
		  count += 1
		  print "%s: %s" % ( threadName, time.ctime(time.time()) )
	
	# Create two threads as follows
	try:
	   thread.start_new_thread( print_time, ("Thread-1", 2, ) )
	   thread.start_new_thread( print_time, ("Thread-2", 4, ) )
	except:
	   print "Error: unable to start thread"
	
	while 1:
	   pass



以上代码执行后，将返回以下结果

    Thread-1: Thu Jan 22 15:42:17 2018
    Thread-1: Thu Jan 22 15:42:19 2018
    Thread-2: Thu Jan 22 15:42:19 2018
    Thread-1: Thu Jan 22 15:42:21 2018
    Thread-2: Thu Jan 22 15:42:23 2018
    Thread-1: Thu Jan 22 15:42:23 2018
    Thread-1: Thu Jan 22 15:42:25 2018
    Thread-2: Thu Jan 22 15:42:27 2018
    Thread-2: Thu Jan 22 15:42:31 2018
    Thread-2: Thu Jan 22 15:42:35 2018
    

虽然它对于低级线程非常有效，但与新的线程模块相比，线程模块非常有限。

## 线程模块 ##
Python 2.4或以上中包含的较新的线程模块为线程提供了比前一节中讨论的线程模块更强大，更高级的支持。

线程模块公开了线程模块的所有方法，并提供了一些额外的方法 - 

- threading.activeCount（） - 返回活动的线程对象数。

- threading.currentThread（） - 返回调用者线程控件中的线程对象数。

- threading.enumerate（） - 返回当前活动的所有线程对象的列表。

除了这些方法之外，threading模块还有Thread类来实现线程。 Thread类提供的方法如下 - 

- run（） - run（）方法是线程的入口点。

- start（） - start（）方法通过调用run方法启动一个线程。

- join（[time]） - join（）等待线程终止。

- isAlive（） - isAlive（）方法检查线程是否仍在执行。

- getName（） - getName（）方法返回线程的名称。

- setName（） - setName（）方法设置线程的名称。

## 使用线程模块创建线程 ##
要使用线程模块实现新线程，您必须执行以下操作 - 

- 定义Thread类的新子类。

- 重写__init __（self [，args]）方法以添加其他参数。

- 然后，重写run（self [，args]）方法以实现线程在启动时应该执行的操作。

一旦创建了新的Thread子类，就可以创建它的一个实例，然后通过调用start（）来启动一个新线程，start（）又调用run（）方法。

### 示例 ###

	#!/usr/bin/python
	
	import threading
	import time
	
	exitFlag = 0
	
	class myThread (threading.Thread):
	   def __init__(self, threadID, name, counter):
	      threading.Thread.__init__(self)
	      self.threadID = threadID
	      self.name = name
	      self.counter = counter
	   def run(self):
	      print "Starting " + self.name
	      print_time(self.name, 5, self.counter)
	      print "Exiting " + self.name
	
	def print_time(threadName, counter, delay):
	   while counter:
	      if exitFlag:
	         threadName.exit()
	      time.sleep(delay)
	      print "%s: %s" % (threadName, time.ctime(time.time()))
	      counter -= 1
	
	# Create new threads
	thread1 = myThread(1, "Thread-1", 1)
	thread2 = myThread(2, "Thread-2", 2)
	
	# Start new Threads
	thread1.start()
	thread2.start()
	
	print "Exiting Main Thread"

以上代码执行后，将返回以下结果


    Starting Thread-1
    Starting Thread-2
    Exiting Main Thread
    Thread-1: Thu Mar 21 09:10:03 2018
    Thread-1: Thu Mar 21 09:10:04 2018
    Thread-2: Thu Mar 21 09:10:04 2018
    Thread-1: Thu Mar 21 09:10:05 2018
    Thread-1: Thu Mar 21 09:10:06 2018
    Thread-2: Thu Mar 21 09:10:06 2018
    Thread-1: Thu Mar 21 09:10:07 2018
    Exiting Thread-1
    Thread-2: Thu Mar 21 09:10:08 2018
    Thread-2: Thu Mar 21 09:10:10 2018
    Thread-2: Thu Mar 21 09:10:12 2018
    Exiting Thread-2
    

## 同步线程 ##
Python提供的线程模块包含一个易于实现的锁定机制，允许您同步线程。 通过调用Lock（）方法创建一个新锁，该方法返回新锁。

新锁对象的获取（阻塞）方法用于强制线程同步运行。 可选的阻塞参数使您可以控制线程是否等待获取锁定。

如果阻塞设置为0，则如果无法获取锁定，则线程立即返回0值，如果获取了锁定，则返回1。 如果阻塞设置为1，则线程阻塞并等待锁被释放。

新锁对象的release（）方法用于在不再需要时释放锁。

### 示例 ###

	#!/usr/bin/python
	
	import threading
	import time
	
	class myThread (threading.Thread):
	   def __init__(self, threadID, name, counter):
	      threading.Thread.__init__(self)
	      self.threadID = threadID
	      self.name = name
	      self.counter = counter
	   def run(self):
	      print "Starting " + self.name
	      # Get lock to synchronize threads
	      threadLock.acquire()
	      print_time(self.name, self.counter, 3)
	      # Free lock to release next thread
	      threadLock.release()
	
	def print_time(threadName, delay, counter):
	   while counter:
	      time.sleep(delay)
	      print "%s: %s" % (threadName, time.ctime(time.time()))
	      counter -= 1
	
	threadLock = threading.Lock()
	threads = []
	
	# Create new threads
	thread1 = myThread(1, "Thread-1", 1)
	thread2 = myThread(2, "Thread-2", 2)
	
	# Start new Threads
	thread1.start()
	thread2.start()
	
	# Add threads to thread list
	threads.append(thread1)
	threads.append(thread2)
	
	# Wait for all threads to complete
	for t in threads:
	    t.join()
	print "Exiting Main Thread"

以上代码执行结果


	Starting Thread-1
	Starting Thread-2
	Thread-1: Thu Mar 21 09:11:28 2018
	Thread-1: Thu Mar 21 09:11:29 2018
	Thread-1: Thu Mar 21 09:11:30 2018
	Thread-2: Thu Mar 21 09:11:32 2018
	Thread-2: Thu Mar 21 09:11:34 2018
	Thread-2: Thu Mar 21 09:11:36 2018
	Exiting Main Thread


## 多线程优先级队列 ##
Queue模块允许您创建一个可以容纳特定数量项目的新队列对象。 有以下方法来控制队列 - 

- get（） - get（）从队列中删除并返回一个项目。

- put（） - put将项添加到队列中。

- qsize（） - qsize（）返回当前队列中的项目数。

- empty（） - 如果队列为空，则empty（）返回True; 否则，错误。

- full（） - 如果队列已满，则full（）返回True; 否则，错误。

### 示例 ###
	
    #!/usr/bin/python
    
    import Queue
    import threading
    import time
    
    exitFlag = 0
    
    class myThread (threading.Thread):
       def __init__(self, threadID, name, q):
      threading.Thread.__init__(self)
      self.threadID = threadID
      self.name = name
      self.q = q
       def run(self):
      print "Starting " + self.name
      process_data(self.name, self.q)
      print "Exiting " + self.name
    
    def process_data(threadName, q):
       while not exitFlag:
      queueLock.acquire()
     if not workQueue.empty():
    data = q.get()
    queueLock.release()
    print "%s processing %s" % (threadName, data)
     else:
    queueLock.release()
     time.sleep(1)
    
    threadList = ["Thread-1", "Thread-2", "Thread-3"]
    nameList = ["One", "Two", "Three", "Four", "Five"]
    queueLock = threading.Lock()
    workQueue = Queue.Queue(10)
    threads = []
    threadID = 1
    
    # Create new threads
    for tName in threadList:
       thread = myThread(threadID, tName, workQueue)
       thread.start()
       threads.append(thread)
       threadID += 1
    
    # Fill the queue
    queueLock.acquire()
    for word in nameList:
       workQueue.put(word)
    queueLock.release()
    
    # Wait for queue to empty
    while not workQueue.empty():
       pass
    
    # Notify threads it's time to exit
    exitFlag = 1
    
    # Wait for all threads to complete
    for t in threads:
       t.join()
    print "Exiting Main Thread"
    
以上代码的执行结果

    Starting Thread-1
    Starting Thread-2
    Starting Thread-3
    Thread-1 processing One
    Thread-2 processing Two
    Thread-3 processing Three
    Thread-1 processing Four
    Thread-2 processing Five
    Exiting Thread-3
    Exiting Thread-1
    Exiting Thread-2
    Exiting Main Thread
    
# END #