---
title: '[leetcode][Python][Concurrency][Medium] 1115. Print FooBar Alternately'
date: '2022-12-24T04:07:08.369Z'
categories: ['leetcode']
keywords: ['concurrency', 'python']
---

## Description

Suppose you are given the following code:
```java
class FooBar {  
  public void foo() {  
    for (int i = 0; i < n; i++) {  
      print("foo");  
    }  
  }

  public void bar() {  
    for (int i = 0; i < n; i++) {  
      print("bar");  
    }  
  }  
}
```

The same instance of `FooBar` will be passed to two different threads:

*   thread `A` will call `foo()`, while
*   thread `B` will call `bar()`.

Modify the given program to output `"foobar"` `n` times.

## Idea

An example for output
```pre
Input: n = 2  
Output: "foobarfoobar"  
Explanation: "foobar" is being output 2 times.
```
Using a flag to switch `printFoo()` and `printBar()` when acquire the lock.

## Solution
```python
from threading import Condition  
  
class FooBar:  
    def __init__(self, n):  
        self.n = n  
        self._lock = Condition()  
        self._is_printed_foo = False  
  
    def foo(self, printFoo: 'Callable[[], None]') -> None:  
          
        for i in range(self.n):  
            with self._lock:  
                while self._is_printed_foo:  
                    self._lock.wait()  
                # printFoo() outputs "foo". Do not change or remove this line.  
                printFoo()  
                self._is_printed_foo = True  
                self._lock.notify_all()  
  
  
    def bar(self, printBar: 'Callable[[], None]') -> None:  
          
        for i in range(self.n):  
            with self._lock:  
                while not self._is_printed_foo:  
                    self._lock.wait()  
                # printBar() outputs "bar". Do not change or remove this line.  
                printBar()  
                self._is_printed_foo = False  
                self._lock.notify_all()

```