---
title: '[leetcode][Python][Concurrency][Medium] 1226. The Dining Philosophers'
date: '2022-12-24T15:05:27.192Z'
categories: ['leetcode']
keywords: ['concurrency', 'python']
showToc: true
TocOpen: true
---

## Description

Five silent philosophers sit at a round table with bowls of spaghetti. Forks are placed between each pair of adjacent philosophers.

Each philosopher must alternately think and eat. However, a philosopher can only eat spaghetti when they have both left and right forks. Each fork can be held by only one philosopher and so a philosopher can use the fork only if it is not being used by another philosopher. After an individual philosopher finishes eating, they need to put down both forks so that the forks become available to others. A philosopher can take the fork on their right or the one on their left as they become available, but cannot start eating before getting both forks.

Eating is not limited by the remaining amounts of spaghetti or stomach space; an infinite supply and an infinite demand are assumed.

Design a discipline of behaviour (a concurrent algorithm) such that no philosopher will starve; _i.e._, each can forever continue to alternate between eating and thinking, assuming that no philosopher can know when others may want to eat or think.

![](/images/leetcode/concurrency/the-dining-philosophers/image_0.png)
_The problem statement and the image above are taken from_ [_wikipedia.org_](https://en.wikipedia.org/wiki/Dining_philosophers_problem)

The philosophers’ ids are numbered from `0` to `4` in a `clockwise` order. Implement the function `void wantsToEat(philosopher, pickLeftFork, pickRightFork, eat, putLeftFork, putRightFork)` where:

*   `philosopher` is the id of the philosopher who wants to eat.
*   `pickLeftFork` and `pickRightFork` are functions you can call to pick the corresponding forks of that philosopher.
*   `eat` is a function you can call to let the philosopher eat once he has picked both forks.
*   `putLeftFork` and `putRightFork` are functions you can call to put down the corresponding forks of that philosopher.
*   The philosophers are assumed to be thinking as long as they are not asking to eat (the function is not being called with their number).

Five threads, each representing a philosopher, will simultaneously use one object of your class to simulate the process. The function may be called for the same philosopher more than once, even before the last call ends.

## Idea

The [Dining philosophers problem](https://en.wikipedia.org/wiki/Dining_philosophers_problem).

In computer science, the dining philosophers problem is an example problem often used in concurrent algorithm design to illustrate synchronization issues and techniques for resolving them.

Focus on the `forks` instead of `philosophers`, because forks are necessary resources if philosopher would like to eat food.

I used a list to put 5 lock, each lock indicates a fork, let `philosopher id + 1` as their `left-hand`, `philosopher id` as their `right-hand`.

Take the `pickup left-hand’s fork first` because it’s have a `higher number` , and `put down right fork` first because it a `higher number fork for right side philosopher`.

## Solution
```python
from threading import Condition  
  
class DiningPhilosophers:  
  
    def __init__(self) -> None:  
        self._forks = [Condition()] * 5  
  
    # call the functions directly to execute, for example, eat()  
    def wantsToEat(self,  
                   philosopher: int,  
                   pickLeftFork: 'Callable[[], None]',  
                   pickRightFork: 'Callable[[], None]',  
                   eat: 'Callable[[], None]',  
                   putLeftFork: 'Callable[[], None]',  
                   putRightFork: 'Callable[[], None]') -> None:  
                     
                   with self._forks[(philosopher+1)%5], self._forks[philosopher]:  
                       pickLeftFork()  
                       pickRightFork()  
                       eat()  
                       putRightFork()  
                       putLeftFork()

```