26题 
semaphore方法：
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
