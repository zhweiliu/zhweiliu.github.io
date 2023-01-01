---
title: '[leetcode][Python][Concurrency][Medium] 1195. Fizz Buzz Multithreaded'
date: '2022-12-24T05:25:42.878Z'
categories: ['leetcode']
keywords: ['concurrency', 'python']
---

## Description

You have the four functions:

*   `printFizz` that prints the word `"fizz"` to the console,
*   `printBuzz` that prints the word `"buzz"` to the console,
*   `printFizzBuzz` that prints the word `"fizzbuzz"` to the console, and
*   `printNumber` that prints a given integer to the console.

You are given an instance of the class `FizzBuzz` that has four functions: `fizz`, `buzz`, `fizzbuzz` and `number`. The same instance of `FizzBuzz` will be passed to four different threads:

*   `Thread A:` calls `fizz()` that should output the word `"fizz"`.
*   `Thread B:` calls `buzz()` that should output the word `"buzz"`.
*   `Thread C:` calls `fizzbuzz()` that should output the word `"fizzbuzz"`.
*   `Thread D:` calls `number()` that should only output the integers.

Modify the given class to output the series `[1, 2, "fizz", 4, "buzz", ...]` where the `ith` token (`1-indexed`) of the series is:

*   `"fizzbuzz"` if `i` is divisible by `3` and `5`,
*   `"fizz"` if `i` is divisible by `3` and not `5`,
*   `"buzz"` if `i` is divisible by `5` and not `3`, or
*   `i` if `i` is not divisible by `3` or `5`.

Implement the `FizzBuzz` class:

*   `FizzBuzz(int n)` Initializes the object with the number `n` that represents the length of the sequence that should be printed.
*   `void fizz(printFizz)` Calls `printFizz` to output `"fizz"`.
*   `void buzz(printBuzz)` Calls `printBuzz` to output `"buzz"`.
*   `void fizzbuzz(printFizzBuzz)` Calls `printFizzBuzz` to output `"fizzbuzz"`.
*   `void number(printNumber)` Calls `printnumber` to output the numbers.

## Idea

For example
```pre
Input: n = 15  
Output: [1,2,"fizz",4,"buzz","fizz",7,8,"fizz","buzz",11,"fizz",13,14,"fizzbuzz"]
```
Basically, I guess it could be used `Condition` or `Lock` to solve this question, but its could be bring about not easily to read for the solution.

After study the discussion with other contributors, I agree to use Python `threading.Semaphore` to solve this question.

The [`Semaphore`](https://docs.python.org/3/library/threading.html#semaphore-objects) introduce on official documentation as :
```pre
A semaphore manages an atomic counter representing the number of   
release() calls minus the number of acquire() calls, plus an initial value.   
  
The acquire() method blocks if necessary until it can return without   
making the counter negative. If not given, value defaults to 1.
```
We can create Semaphore objects for `fizz`, `buzz`, `fizzbuzz` and `numbers`. And use the for-loops setup their runtimes with fit conditions to `n` .

The semaphore initial values are `0` for `fizz`, `buzz`, `fizzbuzz`, but setup the semaphore initial value `1` for `numbers`, because we know the serial start with a number, `1` to `n` , and all conditions of `fizz`, `buzz`, `fizzbuzz` requires divisible by `number` ,at least `3` , it will help the function number to print numbers without `blocking`.

## Solution
```python
from threading import Semaphore  
  
class FizzBuzz:  
    def __init__(self, n: int):  
        self.n = n  
        self._lock_fz = Semaphore(0)  
        self._lock_bz = Semaphore(0)  
        self._lock_fzbz = Semaphore(0)  
        self._lock_num = Semaphore(1)  
  
    # printFizz() outputs "fizz"  
    def fizz(self, printFizz: 'Callable[[], None]') -> None:  
        for i in range(self.n//3-self.n//15):  
            self._lock_fz.acquire()  
            printFizz()  
            self._lock_num.release()  
  
    # printBuzz() outputs "buzz"  
    def buzz(self, printBuzz: 'Callable[[], None]') -> None:  
        for i in range(self.n//5-self.n//15):  
            self._lock_bz.acquire()  
            printBuzz()  
            self._lock_num.release()  
  
    # printFizzBuzz() outputs "fizzbuzz"  
    def fizzbuzz(self, printFizzBuzz: 'Callable[[], None]') -> None:  
        for _ in range(self.n//15):  
            self._lock_fzbz.acquire()  
            printFizzBuzz()  
            self._lock_num.release()  
  
    # printNumber(x) outputs "x", where x is an integer.  
    def number(self, printNumber: 'Callable[[int], None]') -> None:  
        for i in range(1, self.n+1):  
            self._lock_num.acquire()  
            if i%3==0 and i%5==0:  
                self._lock_fzbz.release()  
            elif i%3==0:  
                self._lock_fz.release()  
            elif i%5==0:  
                self._lock_bz.release()  
            else:  
                printNumber(i)      
                self._lock_num.release()

```