# 同步互斥(lec 18) spoc 思考题


- 有"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的ucore_code和os_exercises的git repo上。

## 个人思考题

### 基本理解
 - 什么是信号量？它与软件同步方法的区别在什么地方？
 - 什么是自旋锁？它为什么无法按先来先服务方式使用资源？
 - 下面是一种P操作的实现伪码。它能按FIFO顺序进行信号量申请吗？
```
 while (s.count == 0) {  //没有可用资源时，进入挂起状态；
        调用进程进入等待队列s.queue;
        阻塞调用进程;
}
s.count--;              //有可用资源，占用该资源； 
```

> 参考回答： 它的问题是，不能按FIFO进行信号量申请。
> 它的一种出错的情况
```
一个线程A调用P原语时，由于线程B正在使用该信号量而进入阻塞状态；注意，这时value的值为0。
线程B放弃信号量的使用，线程A被唤醒而进入就绪状态，但没有立即进入运行状态；注意，这里value为1。
在线程A处于就绪状态时，处理机正在执行线程C的代码；线程C这时也正好调用P原语访问同一个信号量，并得到使用权。注意，这时value又变回0。
线程A进入运行状态后，重新检查value的值，条件不成立，又一次进入阻塞状态。
至此，线程C比线程A后调用P原语，但线程C比线程A先得到信号量。
```

### 信号量使用

 - 什么是条件同步？如何使用信号量来实现条件同步？
 - 什么是生产者-消费者问题？
 - 为什么在生产者-消费者问题中先申请互斥信息量会导致死锁？

### 管程

 - 管程的组成包括哪几部分？入口队列和条件变量等待队列的作用是什么？
 - 为什么用管程实现的生产者-消费者问题中，可以在进入管程后才判断缓冲区的状态？
 - 请描述管程条件变量的两种释放处理方式的区别是什么？条件判断中while和if是如何影响释放处理中的顺序的？

### 哲学家就餐问题

 - 哲学家就餐问题的方案2和方案3的性能有什么区别？可以进一步提高效率吗？

### 读者-写者问题

 - 在读者-写者问题的读者优先和写者优先在行为上有什么不同？
 - 在读者-写者问题的读者优先实现中优先于读者到达的写者在什么地方等待？
 
## 小组思考题

1. （spoc） 每人用python threading机制用信号量和条件变量两种手段分别实现[47个同步互斥问题](07-2-spoc-pv-problems.md)中的一题。向勇老师的班级从前往后，陈渝老师的班级从后往前。请先理解[]python threading 机制的介绍和实例](https://github.com/chyyuu/ucore_lab/tree/master/related_info/lab7/semaphore_condition)

2. (spoc)设计某个方法，能够动态检查出对于两个或多个进程的同步互斥问题执行中，没有互斥问题，能够同步等，以说明实现的正确性。


26题 读写者问题
semaphore方法：
```
#coding=utf-8
import threading  
import random  
import time  
#global count

ReaderCount = 0;  
book = 0;

class Writer(threading.Thread):  
    """class using semaphore"""  

    def __init__(self,threadName,CountSem,WriteSem, SleepTime):  
       
       """initialize thread"""  

       threading.Thread.__init__(self,name=threadName)  
       #self.sleepTime = random.randrange(1,6)  
       
       #set the semaphore as a data attribute of the class  
       
       self.CountSemaphore = CountSem
       self.WriteSemaphore = WriteSem
       self.sleepTime = SleepTime


   
    def run(self):  
       """Print message and release semaphore"""  
       global book;
       #acquire write mutex
       print " Thread %s  is waiting for writing!" % (self.getName()) 
       self.WriteSemaphore.acquire() 
       print " Thread %s  is writing!" % (self.getName()) 
       book += 100
       time.sleep(self.sleepTime)
       print " Thread %s  is writing finish!" % (self.getName()) 
       
       #release the  write mutex  
       self.WriteSemaphore.release()  

class Reader(threading.Thread):  
    """class using semaphore"""  

    def __init__(self,threadName,CountSem,WriteSem,ReadSem,SleepTime):  

      """initialize thread"""  


      threading.Thread.__init__(self,name=threadName)  
      #self.sleepTime=random.randrange(1,6) 
      #self.sleepTime = random.randrange(1,6)  
       
      #set the semaphore as a data attribute of the class  
       
      self.CountSemaphore = CountSem
      self.WriteSemaphore = WriteSem
      self.ReadSemaphore = ReadSem
      self.sleepTime = SleepTime

    def run(self):
      #acquire write mutex
      global ReaderCount
      global book
      
      self.CountSemaphore.acquire() 
      if ReaderCount == 0:
        print " Thread %s  is waiting for  WriteSemaphore!" % (self.getName()) 
        self.WriteSemaphore.acquire()
      ReaderCount = ReaderCount + 1
      self.CountSemaphore.release()
      
      print " Thread %s  is waiting for  ReadSemaphore!" % (self.getName()) 
      self.ReadSemaphore.acquire()

      print " Thread %s  is reading! book is %d now" % (self.getName(),book)
      time.sleep(self.sleepTime)
      print " Thread %s  is reading finish!" % (self.getName())

      self.CountSemaphore.acquire()
      ReaderCount = ReaderCount - 1
      if (ReaderCount == 0):
          self.WriteSemaphore.release(); 
      #release the  write mutex  
      self.CountSemaphore.release()  
      self.ReadSemaphore.release()


if __name__ == "__main__":  
    threads = []   

    threadWriteSem = threading.Semaphore(1)
    threadCountSem = threading.Semaphore(1)
    threadReadSem = threading.Semaphore(1)

    threads.append(Writer("thread"+str(2),threadCountSem,threadWriteSem,2.8))
    threads.append( Reader("thread"+str(1), threadCountSem, threadWriteSem, threadReadSem, 3.2) )
    threads.append(Reader("thread"+str(3),threadCountSem,threadWriteSem,threadReadSem,3.1 ))
    threads.append(Reader("thread"+str(4),threadCountSem,threadWriteSem,threadReadSem,3 ))
       #创建一个列表，该列表由SemaphoreThread对象构成，start方法开始列表中的每个线程 
    print len(threads)
    for thread in threads: 
      thread.start() 
```
