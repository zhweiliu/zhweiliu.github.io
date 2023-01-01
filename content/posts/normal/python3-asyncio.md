---
title: 'Python3 - asyncio'
date: '2022-12-22T14:12:05.020Z'
categories: ['Python3', 'Concurrency']
keywords: ['Python3', 'Concurrency', 'asyncio']
---

asyncio is a library to write concurrent code using the async/await syntax.  
  
---- from Python3.11.1 documentation

This article is write down the note with my study of python asyncio package.

## How does _asyncio_ work ?

![](/images/normal/python3-asyncio/image_0.png)
1.  The main process, which is start run by IDE or command line, have a main thread to execute submit a coroutine to asyncio event loops by `asyncio.create_task()` or `asyncio.run()` , the keyword `async` will packet methods as coroutine.
2.  The event loops will monitoring all of the submit task, and choice a not finished, current can going on coroutine to execute its task until it finish, or change status to wait when meet `await` .
3.  When event loops meet `await` , you should be notify (or notify all) task(s) which status is waiting for blocking, and check the blocking condition is still exist or not.
4.  Repeating above step 2 and step 3 until no more coroutines in asyncio event loops (i.e. all of the coroutine will be finish or canceled).

## async & await

*   Using `async` to create a coroutine method
*   Using `await` to call another coroutine `_B_` in coroutine `_A_`.  
     `_A_` will into wait status until `_B_` execute finish and notify.
*   Using `asyncio.run()` to submit a coroutine

![](/images/normal/python3-asyncio/image_1.png)
## Create Task & Submit Coroutine

*   The `asyncio.create_task()` will submit a coroutine into Task Queue
*   When `await` `_task_1_` be execute, main thread will keep waiting until `_task_1_` `finish`/`await`, and so on `_task_2_`
*   It have concurrency effect like as multi threading( or multi processing )

![](/images/normal/python3-asyncio/image_2.png)
## Timeout & Cancel

*   Using `Task.done()` to determine a task is finish or not yet.
*   Using `Task.cancel()` to cancel a task which is executing.

![](/images/normal/python3-asyncio/image_3.png)
*   Using `asyncio.wait_for(task, timeout=wait_duration)` for automate cancel a task if execute timeout

![](/images/normal/python3-asyncio/image_4.png)
Sometimes, we want the task keep going on their work until finish, and I just would like to know the task will happened timeout or not. For example: Counting the times of timeout to calculate performance

## Parallel multi task

*   Using `asyncio.gather(task1, task2, …, taskN)`
*   Add parameter `return_exceptions = True` capture exception result

![](/images/normal/python3-asyncio/image_5.png)