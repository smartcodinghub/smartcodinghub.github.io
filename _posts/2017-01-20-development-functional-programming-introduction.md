---
layout: post
title: '[Development] What is Functional Programming: Introduction'
---

> This series of articles are a product of my personal research on functional programming. 
I will try to simplify all the concepts without losing the original meaning in the way.
To help the transition between traditional programming and functional one, I will use C# at least
in the first few post.
Let's try it!

Functional programming is a paradigm focused on flow, expressions/operations, 
stateless and declarative programming.
This will give you an easier and faster understanding on what's going on, 
you won't have to care about side effects and, of course, predictability.
But, what all these concepts means?

<!--more-->

> I will deep more in the next posts, remember that this is an Introduction!

## Expressions

Expressions are to functional programming what statements are to imperative programming.
An expressions receives an input and produces and output by applying an operation.
Like ``(a, b) => a + b``, which takes two values and produces its sum by applying the
``+`` operator. Here comes some concepts: functions, lambdas, pure functions and 
high order functions.

## Flow

Why I said this paradigm is focused in flow? Well, the flow is tipically part
of the syntax in functional langagues. Its like a chain of calls or pipeline. We start 
with the input, apply a function, the returned value of that functions act as the input 
of the next one and so on. Example:

Where f means a function: 

``f3(f2(f1(f(input))))`` and the "fluent" way: ``input.f().f1().f2().f3()``

A more meaningful example:

``sum(halve(pow(4, 2)), 5)`` and ``4.pow(2).halve().sum(5)``

In either way of doing it, you see clearly the flow that the program will follow. 

## Stateless and side effects

With stateless I mean that the functions won't behave differently based on a state 
with the same input and the data is not mutable. And won't have 
side effects (modify something outside its scope) or at least minize the side effects.
Doing it this way, we ensure that the order of evaluation doesn't matters, 
so you don't need a context to understand whats going on, and 
make the program highly parallelizable.

## Conclusion

We just saw some concepts of Functional programming. Now, we know that it provides
simple, predictable, understable, parallelizable and extendible. But how? We will
see how can we apply all this concepts in C# or any other OOP language and take
advantage in a pragmatic way.

I'm pretty sure that If you had previous knowledge about functional programming,
you are missing some concepts like monads, pure or high-order functions and lazyness but...
Give me time to go deeper!
