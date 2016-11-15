<!-- 
.. title: Let's talk about big D
.. slug: lets-talk-about-big-d
.. date: 2016-11-15 22:14:54 UTC+01:00
.. tags: mindi, c#, philosophy 
.. category: programming 
.. link: 
.. description: 
.. type: text
-->

I would like to start a serie of posts about the context-oriented dependency injection and IoC in C#. This is related to my IoC container framework called **MinDI**. Before introducing MinDI and talking about usage and implementation, let's take briefly theoretical and philosophycal aspects of it. 

## Coupling

I suppose you are familiar with the [SOLID](http://goo.gl/9b6xB6) principles. If not yet, go now and read, and apply it in your programming life, and then come back. SOLID is a must have for any OO design. 

So if you are still here, I guess you also have heard about [IoC containers](http://www.codeproject.com/Articles/615139/An-Absolute-Beginners-Tutorial-on-Dependency-Inver). The IoC container provides generally a nice way to resolve the big D problem of SOLID, and its good implementation encourages and makes it easier to apply all the other biggies as well :) Here we will talk about some aspects of Dependency Inversion and Injection, and about IoC containers, and Context-oriented programming and more.

So, to begin, the largest problem that arise in a more or less complex project, is dependencies and coupling. Generally we should try to minimize the both between modules as much as possible. 

The tight coupling means that class A directly knows class B. This doesn't allow the class A to be tested independently of clase B, not it allows to mock class B, and also if we need a different implementation of class B, we cannot easily substitute it. That also violates big O principle for class A, as it will require changes during the project liftime. That's why big D says, that entities should not depend on each other, but depend on abstractions. This abstraction in C# is called *interface*. Let's see a few examples of coupling:

1. Tight coupling where class A directly knows class B:
```csharp
    class B {
        ...
    }

    class A {
        B b;
        void Action() {
            doSomethingWith(b);
        }
    }
```

As described above, this structure creates major problem and violates SOLID. That's why any medium and large projects that is using tight coupling, become insupportable quite fast.

2. Another example of very bad using of tight coupling is Singletone anti-pattern.
```csharp
class A {
    void Action() {
        B.Instance.DoSomething();
    }
}

```

This is terrible, because in such code many classes start to use singletones, then singletones use other singletones, then everything is tight coupled and explodes when you need to change something. Unfortunately such a bad code can be encountered quite often, especially in the game development, where the quaility of the programming can be quite low. 

#### Next:

- Example of loosy coupling and it's advantages
- Strict interface programming (alsoopen for testing and mocking)
- Defining dependencies in one place
- Introducing IoC container
- Problems that IoC containers have
    * Access container itself
    * Unrestricted access to all the interfaces
    * Handling complex data structures
    * Refactoring friendly

#### Next articles:

# Introducing context as IoC container
    (abstract, concrete, layers, closed context, factories, property injection)
    How do we think

# Usage of MinDI
    







