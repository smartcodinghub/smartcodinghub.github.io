---
layout: post
title: '[Development] What is Functional Programming: Introduction'
---

> This series of articles are a product of my personal research on functional programming. 
I will try to simplify all the concepts without losing the original meaning in the way.
To help the transition between traditional programming and functional one, I will use C# at least
in the first few post.
Let's try it!

<!--more-->

## First, some context:

For a little context, we need to see the history of the paradigms. 

First, we have **Machine Code**. You have tons of instructions and near 0 reusability
and it's so hard to understand.

> **Structured Programming** arrives!

Over machine code we have **Structured/Procedural programming**. An abstraction
over the machine code. We got methods we can easily reuse and we
don't have to worry about machine code and many operations since the compiler
and the language APIs will take care of it. But, we can still taht the code looks similar.

An example:

```csharp
enum PrintEnum { Console, Logger }

public void static Process(int max, PrintEnum pe)
{
    /* We start a loop, with its counter */
    for(int i = 0; i < max; i++)
    {
        /* We must say what Print way want */
        switch(pe)
        {
            case PrintEnum.Console:
                PrintConsole(i * i);
                break;
            case PrintEnum.Logger:
                PrintLogger(i * i);
                break;
        }
    }
}

public static void PrintConsole(int i)
{
    /* We say how we print it */
    Console.WriteLine(i);
}

public static void PrintLogger(int i)
{
    /* We say how we print it */
    Logger.Print(i);
}

...
Process(20);
...
```

> That had no **class**

Then, we started using **OOP** were we bring a new level of abstraction. Here we got classes,
objects and behaviour. We use this to encapsulate and easy reuse all what we do. We can use inheritance
and many other thigs. You all know.

Example:

```csharp
interface IPrinter { void Print(int i); }

public void Process(int max, IPrinter printer)
{
    /* We start a loop, with its counter */
    for(int i = 0; i < max; i++)
    {
        /* We use and object that implements IPrinter to print it (DI here) */
        printer.Print(i * i);
    }
}

...
Process(20, new ConsolePrinter());
...
...
Process(20, new LoggerPrinter());
...
```

> Too much boilerplate code for a little gain?

And what comes after OOP? Here comes the **Functional Programming** (FP). I don't see it as a
subtitution of OOP rather than a _**complement**_. FP is highly focused
in flows. By using FP you are making your aplication/program **modular, extensible, less error prone
and stateless**. As you won't have states or side effects, your program si also easy to predict!
Also, you will program in a _**declarative**_ way, rather than a _**imperative**_ one which makes your 
code easier to understand.

The last one:

```csharp
public void Process(int max, Action<int> print)
{
    // We just say what we want, from 0 to max, select the power, and for each, print it.
    Enumerable.Range(0, max).Select(i => i * i).ForEach(print);
}

...
Process(20, (i) => Console.WriteLine(i));
...
/* Or, complementing it with OOP */
...
Process(20, (i) => new ConsolePrinter().Print(i));
...
/* Now, we want to log and print */ 
...
Process(20, (i) => 
{ 
    new LoggerPrinter().Print(i);
    new ConsolePrinter().Print(i);
});
...  
```

> I will deep more in the next posts, remember that this is an **Introduction**!

## Flow

Why I said this paradigm is focused in flow? Well, the flow is tipically part
of the syntax in functional langagues. Its like a chain of calls or pipeline. We start 
with the input, apply a function, the returned value of that functions act as the input 
of the next one and so on. We can use the word Stream too. 

Example:

Where f means a function that works on a collection: 

``f3(f2(f1(f(input))))`` and the "fluent" way: ``input.f().f1().f2().f3()``

With linq we can see things like:

``Enumerable.Range(0, 50).Select(Pow).Select(Halve).ForEach(Print);``

We can see easily what the program do. Generate numbers from 0 to 50,
power, halve and the print them. And we have lazyness, but that's for another post.

## Stateless and side effects

With stateless I mean that the functions won't behave differently based on a state 
with the same input and the data is not mutable. Just like a Math operation. And won't have 
side effects (modify something outside its scope). Doing it this way, we ensure that 
the order of evaluation doesn't matters, so you don't need a context to understand 
whats going on, and make the program highly parallelizable as you won't have 
race conditions.

## Conclusion

We just saw some concepts of Functional programming. Now, we know that it provides
simple, predictable, understable, parallelizable and extendible. But how? We will
see how can we apply all this concepts in C# or any other OOP language and take
advantage in a pragmatic way.

I'm pretty sure that If you had previous knowledge about functional programming,
you are missing some concepts like monads, pure or high-order functions and lazyness but...
Give me time to go deeper!
