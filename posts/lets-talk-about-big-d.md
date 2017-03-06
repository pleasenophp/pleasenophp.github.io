<!-- 
.. title: Let's talk about big D
.. slug: lets-talk-about-big-d
.. date: 2016-11-15 22:14:54 UTC+01:00
.. tags: mindi, c#, agile, philosophy 
.. category: Programming 
.. link: 
.. description: 
.. type: text
-->

I would like to start a serie of posts about the context-oriented dependency injection and IoC in C#. This is related to my IoC container framework called **MinDI**. Before introducing MinDI and talking about usage and implementation, let's take briefly theoretical and philosophycal aspects of it. 

## Coupling

I suppose you are familiar with the [SOLID](http://goo.gl/9b6xB6) principles. If not yet, go now and read, and apply it in your programming life, and then come back. SOLID is a must have for any OO design. 

So if you are still here, I guess you also have at least heard about [IoC containers](http://www.codeproject.com/Articles/615139/An-Absolute-Beginners-Tutorial-on-Dependency-Inver). The IoC container provides generally a nice way to resolve the big D problem of SOLID, and its good implementation encourages and makes it easier to apply all the other biggies as well :) Here and in the next posts we will talk about some aspects of Dependency Inversion and Injection, and about IoC containers, and Context-oriented programming and more.

So, to begin, the largest problem that arises in a more or less complex project, is dependencies and coupling. Generally we should try to minimize the both between modules as much as possible. 

The tight coupling means that class A directly knows class B. This doesn't allow the class A to be tested independently of class B, nor it allows to mock class B, and also if we need a different implementation of class B, we cannot easily substitute it. That also violates big O principle for class A, as it will require changes during the project lifetime. That's why big D says, that entities should not depend on each other, but on abstractions. Such abstraction in C# is called *interface*. Let's see a few examples of coupling:

**a.** Tight coupling where class A directly knows class B:
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

As described above, this structure creates major problem and violates SOLID. Any medium and large projects, that are using tight coupling, become insupportable quite fast.

**b.** Another example of very bad using of tight coupling is Singleton anti-pattern.
```csharp
class A {
    void Action() {
        B.Instance.DoSomething();
    }
}
```

This is terrible, because in such code many classes start to use singletons inside of the obscured calls, then singletons use other singletons, then everything is tight coupled and explodes when you need to change something. Unfortunately such a bad code can be encountered quite often.

## Setting it free

I briefly tocuhed tight coupling, because it's not very interesting, you can find plenty information about it in the internet. And the solution to the mentioned problem can be formulated in simple statements:

- Make all your code depend only on abstractions (in C# it's interfaces), what's called *loose coupling*
- Make a separate code that configures the abstractions, by matching them with concrete implementations

That gives us the following main advantages:

- Concrete implementations of entities can be replaced separately only in one place, without changing the rest of the code. Our code becomes refactoring friendly.
- TDD-friendly too. Now it's very easy to mock separate interfaces, so the entities become very testable individually. 
- The modularity of the code is increased, and the usage of big L, big S, and big I is especially encouraged. We don't need to create big God classes anymore, as we can easily define the concrete dependencies for each entity.
We can easily susbtitute the concrete implementations of the interfaces, keeping all the rest of the code without any knowledge of the fact, that we changed anything, as everything depends on interfaces, not on implementations. Of course, using of IoC/DI patterns cannot guarantee you will start magically write a good code, but it rather opens all the roads to write a nice code, if you understand what you are doing.  
- The dependencies become strictly defined in each concrete implementatons (we will talk about the Dependency Contract later), and not spread implicitly through the code.

As you can see, it's all about Agile development, where we need to make the code, that is equally responsible to new feature requests in the whole lifetime of the project. Where we also want to use TDD to increase the stability of our builds. And where we want to have some fun by making code that looks nice and makes us happy. 

## We need a framework

Using DI and coding to interface approach also opens some questions:

- if we start depending only on interfaces, what is a good way of easily passing those dependencies to concrete entities?
- if we avoid using singleton patterns, what is a good way to have several entity use the same instance? How do we control lifetime of the objects?
- how can we organize a centralized place of defining the concrete implementations for each interface?
- can those implementations be substituted dynamically during runtime as well?
- can different parts of the code use different concrete implementatons of the same abstractions they depend on?

The IoC/DI framework is designed to solve all those problems. Without such a framework we would need to just pass dependencies manually to every created instance everywhere, and that would be very painfull. So without a framework or support from the language part, the DI pattern and coding to interface approach remain good only on the paper. 

C# is an interface-based language, and should encourage people to use interfaces. If you look at the .NET API itself, they have interfaces for everything. But how many code did you usually encounter in C#, that uses interfaces as a base? That's because Microsoft didn't force to use DI in .NET, leaving the choice of DI, or another pattern to developer. You can use Spring.NET, Unity, or other popular DI libraries. But how many C# programmers ever heard about DI? In the game development, where I worked during 6 years, the most of the C# code I've seen was ugly enough. The better situation is in Java world, where there are more traditions on coding standards. But you have to chose the DI library in Java as well, and the choice sometimes is not easy, because most of the modern libraries are not context-oriented, lack some features, or little refactoring-friendly. While there should still be a choice, I believe, the built in DI/IoC should be part of any modern strongly-typed language. I think people should learn SOLID, DRY and DI principles right together with learning OOP at school. But our world sucks. This was a little complaining part. Now it's time to change the situation, and start using advantages of dependency injection if you don't use it yet.

## A few words about Service Locator pattern

It's ugly. Was that enough words?

- Instead of removing tight coupling, it creates a new tight coupling, where all the classes are singleton-like coupled to service locator itself. 
- It doesn't allow Dependency Contract, not allowing to make the dependencies explicit. That is a major advantage of DI, and we talk later about it. 
- It doesn't allow for any easy implementation of context-oriented and layered dependencies (I will reveal more on this topic later).  

So basically, service locator is improved version of a singleton, that still smells. 

I hope I have encouraged you to at least research information about DI/IoC containers, and maybe try some of them. The next articles will start telling about **MinDI** framework and will show some usage examples.  

