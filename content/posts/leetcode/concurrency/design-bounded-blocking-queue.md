---
title: '[leetcode][Python][Concurrency][Medium] 1188. Design Bounded Blocking Queue'
date: '2022-12-22T07:45:19.754Z'
categories: ['leetcode']
keywords: ['concurrency', 'python']
---

## Description

Implement a thread-safe bounded blocking queue that has the following methods:

*   `BoundedBlockingQueue(int capacity)` The constructor initializes the queue with a maximum `capacity`.
*   `void enqueue(int element)` Adds an `element` to the front of the queue. If the queue is full, the calling thread is blocked until the queue is no longer full.
*   `int dequeue()` Returns the element at the rear of the queue and removes it. If the queue is empty, the calling thread is blocked until the queue is no longer empty.
*   `int size()` Returns the number of elements currently in the queue.

Please do not use built-in implementations of bounded blocking queue as this will not be accepted in an interview.

## Idea

Your implementation will be tested using multiple threads at the same time. Each thread will either be a producer thread that only makes calls to the `enqueue` method or a consumer thread that only makes calls to the `dequeue` method. The `size` method will be called after every test case.
```pre
Input:  
3  
4  
["BoundedBlockingQueue","enqueue","enqueue","enqueue","dequeue","dequeue","dequeue","enqueue"]  
[[3],[1],[0],[2],[],[],[],[3]]  
Output:  
[1,0,2,1]  
  
Explanation:  
Number of producer threads = 3  
Number of consumer threads = 4  
  
BoundedBlockingQueue queue = new BoundedBlockingQueue(3);   // initialize the queue with capacity = 3.  
  
queue.enqueue(1);   // Producer thread P1 enqueues 1 to the queue.  
queue.enqueue(0);   // Producer thread P2 enqueues 0 to the queue.  
queue.enqueue(2);   // Producer thread P3 enqueues 2 to the queue.  
queue.dequeue();    // Consumer thread C1 calls dequeue.  
queue.dequeue();    // Consumer thread C2 calls dequeue.  
queue.dequeue();    // Consumer thread C3 calls dequeue.  
queue.enqueue(3);   // One of the producer threads enqueues 3 to the queue.  
queue.size();       // 1 element remaining in the queue.  
  
Since the number of threads for producer/consumer is greater than 1, we do not know how the threads will be scheduled in the operating system, even though the input seems to imply the ordering. Therefore, any of the output [1,0,2] or [1,2,0] or [0,1,2] or [0,2,1] or [2,0,1] or [2,1,0] will be accepted.
```
I guess the blocking means a task cannot going on its work when some condition cannot fit.

In this question, I need to design a bounded blocking queue, the queue have a capacity that means :

*   Cannot `enqueue` if queue have no remain space for element
*   Can not `dequeue` when no more element in the queue

The `block` happened when meet above situation.

While the thread acquire lock, thread must be detect currently status of queue :

*   Waiting for next time notify if queue have not remaining space for `enqueue`
*   Waiting for next time notify if queue have not any element for `dequeue`

## Solution
```python
from threading import Condition  
class BoundedBlockingQueue(object):  
  
    def __init__(self, capacity: int):  
        self.__capacity = capacity  
        self.__lock = Condition()  
        self.__queue = list()  
  
      
    def enqueue(self, element: int) -> None:  
        with self.__lock:  
            while self.size() == self.__capacity:  
                self.__lock.wait()  
            self.__queue.insert(0, element)  
            self.__lock.notify_all()  
              
      
    def dequeue(self) -> int:  
        ret = None  
        with self.__lock:  
            while self.size() == 0:  
                self.__lock.wait()  
            ret = self.__queue.pop()  
            self.__lock.notify_all()  
          
        return ret  
  
    def size(self) -> int:  
        return len(self.__queue)

```