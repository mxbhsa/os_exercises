26题 condition方法


import threading  
import time  
   
has_writer = threading.Condition()  
reader_num_lock = threading.Condition()  
book = 0  
reader_num = 0

class Writer(threading.Thread):  
    def __init__(self):  
        threading.Thread.__init__(self)  
          
    def run(self):  
        global has_writer, book, reader_num  ,reader_num_lock
        while True:  
            if reader_num_lock.acquire():
                if reader_num < 0:  
                    reader_num_lock.release();
                    if has_writer.acquire():
                        book = int(Math.random() * 100);
                        print "Reader(%s):writing, now book:%s" %(self.name, book)  
                        has_writer.notify();
                        has_writer.release();
                        time.sleep(2)  
                    else :

                else:
                    reader_num_lock.release();
          
class Reader(threading.Thread):  
    def __init__(self):  
        threading.Thread.__init__(self)  
          
    def run(self):  
        global has_writer, book, reader_num  ,reader_num_lock
        while True:  
            if has_writer.acquire():  
                has_writer.release();
                reader_num = reader_num +1;
                print "Reader(%s):reading, now book:%s" %(self.name, book)  
                reader_num = reader_num -1;
                has_writer.notify();
                time.sleep(2)  
                  
if __name__ == "__main__":  
    for p in range(0, 2):  
        p = Producer()  
        p.start()  
          
    for c in range(0, 10):  
        c = Consumer()  
        c.start()  
        
        
semaphore方法：
#coding=utf-8
import threading  
import random  
import time  

book = 0

class Writer(threading.Thread):  
    def __init__(self,rsemaphore, wsemaphore):  
        threading.Thread.__init__(self)
        self.writeSemaphone = wsemaphore;  
        self.readSemaphore = rsemaphore;
        self.sleepTime=random.randrange(1,6);

    def run(self):  
        while True:
            self.writeSemaphone.acquire()  
            book = int(Math.random() * 100);
            print "Reader(%s):writing, now book:%s" %(self.name, book)  
            self.writeSemaphone.release();
            sleep(self.sleepTime);

class Reader(threading.Thread):  
    def __init__(self,rsemaphore, wsemaphore):  
        threading.Thread.__init__(self)  
        self.writeSemaphone = wsemaphore;  
        self.readSemaphore = rsemaphore;
        self.sleepTime=random.randrange(1,6);

    def run(self):  
        while True:
            if self.readSemaphore.acquire()  
            book = int(Math.random() * 100);
            print "Reader(%s):writing, now book:%s" %(self.name, book)  
            self.writeSemaphone.release();
            sleep(self.sleepTime);
            
if __name__ == "__main__":  
    for p in range(0, 2):  
        p = Writer()  
        p.start()  
          
    for c in range(0, 10):  
        c = Reader()  
        c.start()  

