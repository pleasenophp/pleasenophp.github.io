<!-- 
.. title: Meet the IoC container
.. slug: meet-the-ioc-container
.. date: 2017-03-06 00:01:21 UTC+01:00
.. tags: mindi, c#, agile, philosophy 
.. category: Programming
.. link: 
.. description: 
.. type: text
-->

In the [previous](lets-talk-about-big-d.html) article there was a general talk about DI. Now let's see the basic problems and principles of the IoC container.

Let's see some common questions, that arise with an IoC container we want to create:

- How will we access the container itself throughout the application? Will it be one singletone? Sounds a bit crap, and similar to Service Locator then. Will there be only a single place where we create all the classes and thus have access to the container? Then how can we easily create new objects and inject dependencies during runtime?
- How are we going to limit the access to different dependencies? Our container is a universal factory, that gives us an access to every interface of the application? Can we limit it, so different classes could have access only to what is defined in their dependency contract? So we avoid implicit dependencies?
- How are we going to inject our dependencies? Using constructor? Using properties? How do we handle the complex graphs with cross-references? 
- How do we define all our dependencies in one place of applications and keep those definitions refactoring-friendly?
- Will we support multiple layers, where we can redefine some of the dependencies for some of the parts in the application?

Different IoC/DI solutions have different approaches to those problems. Here I would like to introduce how it's solved in **MinDI** and to not bore you with more theory, jump to some examples.
MinDI is a IoC/DI framework, that was initially started as a project to extend the [MinIOC](https://bitbucket.org/Baalrukh/minioc/wiki/Home) framework with some syntax sugar, but then quickly turned into its own project, with much more advanced features and ideology. You can read an overview of the MinDI features, structure and history here (TODO - references). 

## Dependency Injection 

The dependencies can be resolved in several ways: passing them in constructor, assigning them to fields / properties of the object, or passing them in a method of the object.
MinDI uses reflection to make such dependency injection automatic. 

There supported two ways of automatic dependency injection:

- Property based
- Method based

The property based way is recommended, though the method based is also supported (basically because it was originally supported in MinIOC). 
The constructor automatic dependency injection is dropped. The reason of it is because the constructor dependency injection doesn't allow us to use complex object-graph with the circular dependencies. Even though the circular dependeincies are most of the time not good, they are quite usable in some data structures (graphs, trees, DOM, etc). 

There is also usability reason why Property-based DI is recommended: it allows to easily specify the dependency contract, and refactor the amount of dependencies. Using constructors or methods, it becomes quite bulky.

So, here is an example of class **Earth** that has dependencies on *ISun* and *IMoon* interfaces:

```csharp
public class Earth : IEarth {
	[Injection] public ISun sun {get; set;}
	[Injection] public IMoon moon {get; set;}
}
```

In this simple example, the **Injection** attribute tells that those dependencies will be resolved authomatically when an instance of *Earth* is created. An exception will occur, if those dependencies could not be resolved. 

As we see, we depend on interfaces, not on the concrete implementations of Sun and Moon. That is an implementation of the big D principle: we depend upon abstractions and the abstraction are resolvedby the IoC framework to the concrete implementations. That means that we can easily configure the concrete implementations our interfaces can be resolved to in the single place of the application, without changing any dependent entities.

Please also pay attention, that the dependencies are declared public. This is not necessary, but is recommended. That is made to make class easily unit-testable. In a unit test we might want to create just an instance of class *Earth*, and inject/mock the sun and moon properties manually. 

As we always depend on abstractions in our application, one should not worry about making those properties inaccessible for client: the client can only access *IEarth* interface, but not the instance of the *Earth* directly. And the *IEarth* interface limits the access to the properties in any necessary way (they might be not exposed, exposed for get only, exposed for both get and set, etc). 
Using interface-based approach when you are programming is very important principle that allows us to totally depend on abstractions only and easily substitute the concrete implementatons when needed. That makes the code very flexible and easily refactorable, and this encourages to use SOLID principles.

Even though MinDI doesn't require to use interface-based approach only, this approach is highly recommended for any program. Interfaces in C# exist exactly for this, but the issue with the C# language is that they have given us interfaces, but didn't give any easy way of using them, without creating extra pain. With a DI framework, using interfaces-based approach turns into an easy walk and pleasure.  

So, to summarize this very important principle: **we should never depend on a class anywhere in the code** (unless it's pure data class or structure). We should always depend on the interfaces. 

Let's now see how and where we specify that *ISun* should resolve to an instance of class Sun and *IMoon* to an instance of class Moon. 

## IoC container and context-oriented approach

The IoC container is basically a dictionary, that has interface type at minimum as a key and the factory that specifies how we create the object for this interface as a value. Additional features of the container is to control the lifetime of the objects. Even more advanced feature, is to provide the multi-layer context for the dependency injection.

Let's see a simple example:

```csharp
public MyContextInitializer : IApplicationContextInitializer {
	public void Initialize(IDIContext context) {
		context.m().Bind<ISun>(() => new Sun());
		context.m().Bind<IMoon>(() => new Moon());
	}
}
```

Here we defined 2 bindings: each injection of *ISun* will resolve itself to new instance of class *Sun*, and the same with *Moon*. Now whenever the new instance of *Earth* is created, the Injection of *ISun* will be resolved to the new *Sun()*, and so on with *IMoon*. Of course for this to work, the class *Earth* itself should be resolved from the same dependency injection container. When any of the classes is created from the context, its dependencies are fulfilled automatically from the same context. So the IoC container is the context of the possible dependencies. Each class provides the set of [Injection] attributes, that is called in MinDI **contract on dependencies**. Like this we can easily see which exactly dependencies this or that class uses. Having an explicit contract is very benefitial when refactoring and analyzing the code, trying to minimize the amount of the entities each class depends on.

As we can see, if we want *ISun* to resolve to some *MySuperSun* instance instead of *Sun*, it's very easy to change it only in one place of the application: in the context initializer. All the objects, that depend on ISun will now use another class as implementor:

context.m().Bind<ISun>(() => new MySuperSun());

## 3 levels of the application

Unlike some other DI frameworks, that use XML to define the dependencies, MinDI uses the lambda-syntax. That allows the code to be easily refactorable. If we rename a class or an interface, it will be automatically reflected in the context initializers.

Let's now talk a little about the access to our context in the application. Unlike the class, which has strictly access to only its own dependencies the IoC container, or **the context** is a universal factory, which can be used to resolve any of the interfaces. To obtain directly any concrete instance using the context it's enough to call the following:

```csharp
var sun = context.Resolve<ISun>();
```

So this code will find a corresponding factory in the context and will create an object. In this example it will be an object of class Sun. If the user classes have direct access to the context, it creates a problems - we can suddenly anywhere in the code resolve any interface, and this creates implicit dependencies, bypassing the contract on dependencies. It becomes impossible to easily say which dependencies are used in the class. So, to avoid this problem, MinDI doesn't allow direct access to the context for the user-level classes. 

The application in MinDI is conditionally divided in 3 levels of the access:

1. **The context initialization level**. This is the place where we initialize the context, like MyContextInitializer. In this place we don't put any logic, but only define which interface is resolved by what class, and also we define the lifetime and some other more advanced things in the scope of the IoC container.
2. **The user level**. This is where all the application classes function. On this level we have no access to the context, but we have dependency contracts in the classes, so all the dependencies are resolved automagically.
3. **Open context level or factory level**. This is special classes that implement different creational patterns - like factories and builders. Such classes have access to the context from one side, and are used by the user-level classes from another side. Such classes should not contain any logic but building other objects. Usually each factory or builder has a *factory contract*, that limits which exactly type of objects it can build. 

So, let's say we want the class Earth to dynamically create some plants, using interface IPlant. It can create many instances of IPlant and it knows nothing about what concrete class will be used for the IPlant interface, as it is defined only on the context initialization level.

// TODO - example of using factories




Controlling lifetime



Standard context layers



In the next article I will give a simple tutorial on how to start using MinDI, from the beginning to advanced features.




<!--
#### Next:

Principles:
- Abstractions are open for dependent classes, but the implementatons are closed
- Context is an abstract factory that resolves abstractions to concrete classes by the name and types
- Context is populated in the application start point, and at the objects building point, and in no other place in the program should the concrete entities be referenced in any way.
- The entity can depend only on the abstractions, that explicitly specified in the contract 
- The entities have no open access to context, or to the concrete implementations,  except entities of creational patterns (factories, builders)
- Any dynamic dependencies should not be resolved implicitly, but should use contracts (factory interfaces)

3 level of the access:
- Full access for context
- Context access for creation patterns
- Abstractions access for the rest of the code

3 main Layers
- Global for library dependencies 
- Application for redefining library dependencies, and defining application dependencies
- Factory, for defining dependencies of dynamically created objects (with lifetime less than application)

Prototype principle of the dependencies.



- Introducing IoC container
- Problems that IoC containers have
    * Access container itself
    * Unrestricted access to all the interfaces
    * Handling complex data structures
    * Refactoring friendly

#### Next articles:

#### Introducing context as IoC container
    (abstract, concrete, layers, closed context, factories, property injection)
    Generics example
    How do we think example, in context

#### Usage of MinDI articles
(simple HW, Unity application, generics application, factory context, constructions, etc)
-->
