---
layout: post
title: '[Development] What is Functional Programming: Functions'
---

A function in functional programming is just as in other paradigms. It's a 
sequence of operations that can be named to easily referenciate them. In example:
    
``int Sum(int a, int b) { return a + b; }``

or

``Func<int, int, int> sum = (a, b) => a + b;``

Here you can see two ways of declaring a Function in C#, the traditional (first one) 
or the functional (second one).The first one is a simple method. The second one is a 
variable that points to an expresion so you can use it like a method. 
It's a little big difference that you will love at the end of this series.

<!--more-->

## Lambdas

Is just another syntax for declaring an anonymous method. It allows you a more compact, meaningfull 
and fast way of doing it by ignoring keywords and the classical syntax. The structure is simple: 
``(param1, param2, ..., paramN) => operation;``

So:

You can write

``int Sum(int a, int b) { return a + b; }``

as

``Func<int, int, int> sum = (a, b) => a + b;``

Here you can see the two parameters between ``()`` named ``a`` and ``b`` then you see the arrow
and finally you can see the operation you want to make with the params: ``a + b``. The lambda is 
just the right part of the ``=`` but in C# we have types and we can infer it from the context.
The context here is the ``Func<int, int, int>`` declaration of sum variable. The first two 
generic parameters are the arguments and the last one is the return type of the function. Other 
languages uses diferent syntax. An example in Python: ``lambda a, b: a + b``

## Pure functions

> The pure functions is what makes the paradigm stateless and have no side effects. But, why?

Pure functions are functions that follow two laws:

1. A pure function is a function **must always** return the same output for the same input.
So you can't use a **state** or any other info to change the output. 

2. A pure function doesn't have side effects such change a mutable object or make some 'I/O'
*(Any 'I/O' operation have side effects and make changes by definition)*.

Some examples are: 

- Operations in Math class like Pow. The sum function of the previous section.
- Hash functions like md5 that can be bruteforced.
- Any constant function like ``Func<int> gConstant = () => 9.8;``

### Benefits

1. Easy and predictable the behaviour.
2. Self descriptive.
3. No SIDE EFFECTS. Side effects are a bit evil... and hard to debug.
4. No need for storing of states.
5. Easy parallelizations. That's because there's no side effects.
6. Thread safety.
7. Memoization or caching results of hard processes.

## High-order and first-class functions

> Now, I want to make ir a bit more difficult. But just a bit more. 

A **high-order** function is a mathematical concept. Wait and don't run yet. It's just a
function that can either take a function as argument or return it. Example:

    public Func<int, int> ApplyManyTimes(Func<int, int> func, int times)
    {
        return (n) => Enumerable.Range(0, times).Aggregate(n, (seed, next) => func(seed));
    }

And we can call this with: 

    /* New Func */
    Func<int, int> unmodifiedFunc = (n) => n * 2;
    Func<int, int> modifiedFunc = ApplyManyTimes(unmodifiedFunc, 3);
    int result = modifiedFunc(2);

The modifiedFunc is will duplicate three times the given number.

**First-class** function is a programming concept that means that you can use a function 
just like a class, object or literal anywhere in the program (like objects in OOP). 
We have been doing this all the time by referring a function with a variable like and object.
Example: ``Action<int> act = (n) => Console.WriteLine(n);``

With this concepts we achieve a new level of abstraction. Also, we gain a lot of composability.
Examples:

#### Composition:

We can join two different methos to be executed always together via a **high-order** functions.

    public Func<T, TReturn> Compose<T, TReturn>(Func<T, TReturn> f1, Func<T, TReturn> f2)
    {
        return (f1, f2) => (n) => f2(f1(n));
    }

    Func<int, int> func1 = (n) => n * 2; // Doubling
    Func<int, int> func2 = (n) => n + n; // Doubling again
    Func<int, int> composed = Compose(f1, f2);

Let me explain this code a bit. ``func1`` and ``func2`` takes and return and integer. 
``Compose`` takes two functions like ``func1`` and ``func2`` and return only one function.
That function will execute ``f1`` and give the result as the argument to ``f2``.  So, if we execute
``composed(2)`` it will return ``8`` (``func1(2)`` = 4, ``func2(4)`` = 8). After composing,
you will can call this function as many times as you want.

Of course, you can do it with the Composition pattern, but you need a lot more of code. And, with
the functional approach, the composed function will have a smaller scope and be trully private.

#### Decoration:

We can wrap a function call with anything we want. Example:

    public Action<T> Decorate<T>(Action<T> action)
    {
        return (arg) =>
        {
            Console.WriteLine("Start log: " + arg);
            action(action);
            Console.WriteLine("End log: " + arg);
        }
    }

    Action<int> action = (n) => Console.WriteLine("Action: " + n);

    /* Before decoration */
    action(2); 
    // "Action: 2"

    /* After decoration */
    action = Decorate(action);
    action(2);
    // "Start log: 2"
    // "Action: 2"
    // "End log: 2"

Here we use a Decorate method which gives us a decorated version of the func we provides.

### Benefits

Well... composing, decorating, expansion of other functions without touch the original code.
Many more, but... this is the theorical, abstract and obscure explanation. But I will give you a practical use 
case in the real world.

Imagine an old and obscure legacy system (Unfortunatly that's so easy, right?). 

#### First case

You found that you need to use a method with some additions many times like:

    Calculator calculator = new Calculator();
    Result result = calculator.CalculateInfo(x, y);
    Result correctedResult = calculator.NewApplyCorrections(x, y, info);

Of course, you can make a method in the Calculator class or you can make an extension method. The First
way doesn't follow the SOLID principles an the second one gives the method a high scope. You only
want to use it in your own method.

    Func<int, int, Result> NewCalculateInfo = (x, y) => 
    {
        Calculator calculator = new Calculator();
        Result result = calculator.CalculateInfo(x, y);
        return calculator.NewApplyCorrections(x, y, info);
    };

Now you can use it as many times as you want, but only in its _scope_. You didn't change the old class
and you don't have to worry about some one using your help method without undertanding it.

#### Second case

