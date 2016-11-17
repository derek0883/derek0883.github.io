---
layout: post 
serial: "From brute force to ultimate optimization - Recursion"
title: "Recursion - The queen of algorithm"
categories: algorithm
comments: true
---

> From brute force to ultimate optimization - Recursion Method. I solve different algorithm problem with one simple strategy -- Recusion, we get clue from here, then optimize the solution step by step.

I love recursion. recursion is simple, recursion is neat, recursion is a beauty, and recursion is art of compute science. On of greatest German mathematician Gauss said: mathematics is the queen of the sciences and number theory is the queen of mathematics. As an ESL – English as a Second Language, please allow me to copycat this wisdom to show my faith to recursion.
**Algorithm is the queen of the computer sciences and recursion is the queen of Algorithm.**

When you were given an algorithm or data structure problem, you might have no clue how to solve it. after you searched online and checked other one's solution. no matter what method they use. e.g. DP, BSF, DFS, Divide-and-Conquer, and so on. you might feel, Oh it is not difficult at all. I should have be able to solve it independently. Then even give you another similar problem, you still have no clue, until you checked answer again, and you experience same feeling again, It is so easy, why couldn't I have thought in that way?
Don't be frustrated, My serial <<From brute force to ultimate optimization (a recursion method)>> is right for you. Most people only focus on final optimized answer, but a much important thing is the journey from unknown to ultimate right answer, how you approach the problem step by step. ancient Chinese says: "Give a man a fish and you feed him for a day; teach a man to fish and you feed him for a lifetime."
keep in mind, fish is not our goal, fishing skill is what we ultimately pursued. This serial will give you fishing skill other than fish.


Fibonacci numbers famous example in any algorithm text-book, I just follow this unwritten rules here to show you the beauty of recursion.
In mathematical terms, the sequence Fn of Fibonacci numbers is defined by the recurrence relation
Fn = Fn-1 + Fn-2
with seed values
F0 = 0 and F1 = 1.

```java 
long fibonacci(int n) {
  if (n <= 1)
    return n;
  else 
    return fibonacci(n-1) + fibonacci(n-2);
}
```

Fore sure, this simple and neat example will not convince you that how powerful recursion it is. Don't worry, as this serial going on, you will be surprised by the beauty of recursion.
