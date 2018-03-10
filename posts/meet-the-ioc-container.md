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

## MinDI

Please refer to GitHub repository to use the framework itself: [MinDI on GitHub](https://github.com/pleasenophp/mindi)

## Introduction

Here are some common questions, that arise with an IoC container we want to create:

- How will we access the container itself throughout the application? Will it be one singletone? Sounds a bit crap, and similar to Service Locator then. Will there be only a single place where we create all the classes and thus have access to the container? Then how can we easily create new objects and inject dependencies during runtime?
- How are we going to limit the access to different dependencies? Our container is a universal factory, that gives us an access to every interface of the application. Can we limit it, so different classes could have access only to what is defined in their dependency contract? So we avoid implicit dependencies?
- How are we going to inject our dependencies? Using constructor? Using properties? How do we handle the complex graphs with cross-references? 
- How do we define all our dependencies in one place of applications and keep those definitions refactoring-friendly?
- Will we support multiple layers, where we can redefine some of the dependencies for some of the parts in the application?

Different IoC/DI solutions have different approaches to those problems. Here I would like to introduce how it's solved in [MinDI](https://github.com/pleasenophp/mindi) and and show some examples.
Please note, that this article is not a tutorial, but rather a methodological description of MinDI library. The tutorials will be posted later. 

MinDI is a IoC/DI framework, that was initially started as a project to extend the [MinIOC](https://bitbucket.org/Baalrukh/minioc/wiki/Home) framework with some syntax sugar, but then quickly turned into its own project, with much more advanced features and ideology. 

## Dependency Injection 

In such languages as C# or Java, the dependencies can be resolved in several ways: passing them in constructor, assigning them to fields / properties of the object, or passing them in a method of the object.
MinDI uses reflection to make such dependency injection automatic. 

There supported two ways of automatic dependency injection:

- Property based
- Method based

The property based way is recommended, though the method based is also supported (basically because it was originally supported in MinIOC). 
The constructor automatic dependency injection is dropped. The reason of it is because the constructor dependency injection doesn't allow us to use complex object-graph with the circular dependencies. Even though the circular dependeincies are most of the time not good, they are quite usable in some data structures (graphs, trees, DOM, etc). 

There is also usability reason why Property-based DI is recommended: it allows to easily specify the dependency contract, and refactor the amount of dependencies in the class. Using constructors or methods, it becomes quite bulky.

So, here is an example of class **Earth** that has dependencies on *ISun* and *IMoon* interfaces:

```csharp
public class Earth : ContextObject, IEarth {
	[Injection] public ISun sun {get; set;}
	[Injection] public IMoon moon {get; set;}
}
```

In this simple example, the **Injection** attribute tells that those dependencies will be resolved automatically when an instance of *Earth* is created. An exception will occur, if those dependencies could not be resolved. 

As we see, we depend on interfaces, not on the concrete implementations of Sun and Moon. That is an implementation of the **big D** principle: we depend upon abstractions and the abstraction are resolved by the IoC framework to the concrete implementations. That means, we can easily configure the concrete implementations of our interfaces in the single place of the application, without changing any dependent entities. That also encourages the programmer to use the **big L** principle: we write classes that don't know anything about the concrete implementations they use, and should still work if we exchange the implementations in the configuration of the application.

Please also pay attention, that the dependencies are declared public. This is not necessary, but is recommended. That is made to make class easily unit-testable. In a unit test we might want to create just an instance of class *Earth*, and inject/mock the sun and moon properties manually. As we always depend on abstractions in our application, one should not worry about making those properties inaccessible for client code: the client can only access *IEarth* interface, but not the instance of the *Earth* directly. And the *IEarth* interface limits the access to the properties in any necessary way (they might be not exposed, exposed for get only, or exposed for both get and set). 
Using interface-based approach in your programs is very important principle that allows us to totally depend on abstractions only and easily substitute the concrete implementatons when needed. That makes the code very flexible and easily refactorable, and this encourages to use SOLID principles.

Even though MinDI doesn't require to use interface-based approach only, this approach is highly recommended for any program. Interfaces in C# exist exactly for this, but the issue with the C# language is that they have given us interfaces but have not provided an easy way of using them, without creating an extra pain. With a DI framework like MinDI, using interfaces-based approach turns into an easy walk and pleasure.  

So, to summarize this very important principle: **we should never depend on a class anywhere in the code** (unless it's pure data class or structure). We should always depend on the interfaces.  

Let's now see how and where we specify that *ISun* should resolve to an instance of class Sun and *IMoon* to an instance of class Moon. 

## IoC container and context-oriented approach

The IoC container, or **the Context**, how it's called in MinDi, is basically a dictionary, that has interface type at minimum as a key and the factory that specifies how we create the object for this interface as a value. Additional features of the container is to control the lifetime of the objects. Even more advanced feature, is to provide the multi-layer context for the dependency injection.

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

As we can see, if we want *ISun* to resolve to some *MySuperSun* instance instead of *Sun*, it's very easy to change this only in one place of the application: in the context initializer. All the objects, that depend on *ISun* will now use another class as implementor:

```csharp
context.m().Bind<ISun>(() => new MySuperSun());
```

Let's now see how we can access the context in the application to resolve our instances, and what special usage and phylosophical meaning the context has.

## 3 levels of the application

Unlike some other DI frameworks, that use XML to define the dependencies, MinDI uses the lambda-syntax. That allows the code to be easily refactorable. If we rename a class or an interface, it will be automatically reflected in the context initializers. Another benefits of lambda-factories, is that instantiating such objects uses new operator, and is much faster, than reflection, that some of the popular DI frameworks use. 

Let's now talk a little about the access to our context in the application. Unlike the class, which has strictly access to only its own dependencies, the the context is a universal factory, which can be used to resolve any of the interfaces used in the application. To obtain directly any concrete instance using the context it's enough to call the following:

```csharp
var sun = context.Resolve<ISun>();
```

So this code will find a corresponding factory in the context and will create an object that implements interface *ISun*. If the user classes have direct access to the context, it creates problems - we can suddenly anywhere in the code resolve any interface, and thus create implicit dependencies, bypassing the contract on dependencies. It becomes impossible to easily say which dependencies are used in the class. So, to avoid this problem, MinDI doesn't allow a direct access to the context for the user-level classes. 

The application in MinDI is conditionally divided in 3 levels of the access:

1. **The context initialization level**. This is the place where we initialize the context, like *MyContextInitializer* in the example above. In this place we don't put any application logic, but only define which interface is resolved by what class, and also we define the lifetime and some other more advanced things in the scope of the IoC container.
2. **The user level**. This is where all the application classes function. On this level we have no access to the context, but we have dependency contracts in the classes, so all the dependencies are resolved automagically.
3. **Open context level or factory level**. This is special classes that implement different creational patterns - like factories and builders. Such classes have access to the context from one side, and are used by the user-level classes from another side. Such classes should not contain any logic but building other objects. Usually each factory or builder has a *factory contract*, that limits which exactly type of objects it can build. 

To demonstrate how the factories work in MinDI, let's see a simple example. Let's say we want the class Earth to dynamically create some plants, using interface IPlant. It can create many instances of IPlant and it knows nothing about what concrete class will be used for the IPlant interface, as it should be defined only on the context initialization level.

So, somewhere in *MyContextInitializer* we define:

```csharp
context.m().Bind<IPlant>(() => new WeedPlant());
```

Now in our class Earth we wanna have some code that spawns 3 plants:

```csharp
public class Earth: ContextObject, IEarth {
	...
	[Injection] public IDIFactory<IPlant> plantFactory {get; set;}
	...

	void CreatePlants() {
		var plant1 = plantFactory.Create();
		var plant2 = plantFactory.Create();
		var plant3 = plantFactory.Create();
		...
	}
	...
}
```

We use a factory instead of creating the *WeedPlant* directly by new operator, which would make our *Earth* class tightly coupled with class WeedPlant. That would violate the **big D**, and make the instance of class Earth always depend on concrete instance of class WeedPlant. With using **IDIFactory** interface provided by MinDI, we easily resolved this problem. Now the class *Earth* knows that it will create some of the instances of *IPlant*, and doesn't know anything about which exactly IPlant implementation is used.

The way we declared IDIFactory<IPlant> is called in MinDI a **contract on creation**. Same as the contract on dependencies explicitly defines which abstractions this class depends on, the contract on creation defines which entities this class can create. For each type of entity this class can potentially create we add one more IDIFactory injection. 

The construction parameters (e.g. the height of the plant)  can be also passed to the factory by several ways in MinDI. This will be shown in the tutorial, and also there will be discussed a bit more of philosophical aspect of passing the parameters when creating an object through abstraction.

Back to the levels of the access, our class *Earth* is an entity that functions on the user level in MinDI. It has no direct access to the context, but it has explicit contracts on dependencies and creation. The class *Earth*, as you maybe noticed is inherited from *ContextObject*. It's a special object that makes auto-injection of dependencies possible on user-level. In fact, this object has the Context reference as a private field, so it works a bit like subconsious: it does the work behind the scenes, but is not accessible for the user level.


## The philosophy of the context-oriented DI, and multi-layered context

Making more analogy with the human mind, the user level is a bit like our thoughts. You can think "plant", and you immediatelly imagine some sort of plant: you have a concrete visual image appearing immediately in your inner screen. In this analogy the word "plant" is an abstraction, an interface. It openly exists on the user-level, and you are directly aware of it. The image of plant is concrete implementation of this interface. Different people will have different images when they think the same word "plant". What exact image you will have when you think this word is the result of your individual association, that sits in your subconsious. You cannot know it until you think or say "plant", but then it appears immediately. The associations are formed in the mind as the result of life experience, and this level is not directly accessible for the "user" - the regular thinking mind. Each person have a different life experience and the different context. The same way the context initialization level in MinDI is an associative array, that is configured in a single place of the application, and is not dirrectly accessible for the user-level classes. However, as soon as we inject *IPlant* somewhere, it's immediately resolved to the concrete implementation (*WeedPlant* in our example).

Now if you want an analogy with the *factory level*, it's more like our imagination or an ability to think with abstractions. If you are in the room, and you have a plant on your table, it's like an [Injection], something that is already there. However, you are able to think about more plants, that don't exist here. This is like an abstract factory, which can create many instances of IPlant.  

Let's see another interesting feature of our mind: our subconsious level works on the context-oriented basis. In a very simple example, if I ask you, "show me a plant", then you will point to the plant, that stays on your table. But if you don't have any, you will point me to the window, where e.g. we can see a bush outside. Here there is an example of multiple levels of context. The context of the room overrides the more global context of the world outside. So you will likely first search for the object in the room context, and only if not found, will search it in more global context. Of course our mind works much more complicated than this, we have many dynamic context layers that have also multiple cross-references, but this simple example helps to understand the context-oriented approach that is used in MinDI: **the dependencies are resolved from the current context of the entity, if not found they are looked in the parent context, and so on, by the chain of prototype**.

That means, that every Context in MinDI can have a reference to the parent context. That allows to create layers in the applications where some of the dependencies are "overriden", but the others are still used from the global context. It works very similar to the reference prorotype paradigm (like e.g. in JavaScript or Lua). 

In standard MinDI application there are 3 main layers of the context (though you can create any number of them).

1. Global layer: usually we define the global dependencies, that are common for the whole family of the applications. It's often the dependencies exported from the shared libraries.
2. Application layer. This is the dependencies, that are particular for this concrete application. 
3. Custom layer. This is the dependencies that work only for a part of application, and their lifetime is less than the lifetime of the application. E.g. we create a window that overrides the behaviours of the keyboard event receiver with its own. Only this window uses different logic for the event receiver, but it doesn't affect the other parts of the application.

There can be also more layers, for example in the ASP.NET we can have a request level of dependencies, that are particular for each web request. In Unity3D application we have a scene layer of dependencies, that exist for every particular scene, and are not valid for others.

Let's see a short practical example. In a dll we have a *Logger* class, that implements *ILogger* interface. This logger uses *ILogMessageWriter* to specify how exactly we write the message.   

```csharp
public interface ILogger {
	void LogDebug(string message);
	void LogError(string message);
}

internal class Logger: ContextObject, ILogger {
	[Injection] public ILogMessageWriter writer {get; set;}

	public void LogDebug(string message) {
		writer.Write("[DEBUG] "+message);
	}

	public void LogError(string message) {
		writer.Write("[ERROR] "+message);
	}
}
```

The standard LogMessageWriter, our library provides, will use Console.WriteLine method:

```csharp
public interface ILogMessageWriter {
	void Write(string message);
}

internal class ConsoleLogMessageWriter : ILogMessageWriter {
	public void Write(string message) {
		Console.WriteLine(message);
	}
}
```

We can bind our classes, library provides, in the global context initializer of the library:

```csharp
public class LoggerLibraryGlobalContextInitializer: IGlobalContextInitializer {
	public void Initialize(IDIContext context) {
		context.m().Bind<ILogger>(() => new Logger());
		context.m().Bind<ILogMessageWriter>(() => new ConsoleLogMessageWriter());
	}
}
```

Now, if we import the dll or code with the logger into our application, the standard bindings will be available: wherever we inject *ILogger* in our application classes, using [Injection] attribute, it will just work, and write messages to the console. We derive from the standard MinDI interface: *IGlobalContextInitializer*, which means that those bindings will be put in the top-level context: the global layer.

Let's say we have a Unity3D application, where we want to output all our log messages not to the console, but to the Debug.Log() method, Unity provides, which will show them in the editor UI. We can do it very easy, withou chanhing the behaviour of any classes that use ILogger. Let's just create a UnityLogMessageWriter in our application:

```csharp
internal class UnityLogMessageWriter : ILogMessageWriter {
	public void Write(string message) {
		Debug.Log(message);
	}
}
```

Now we need to bind it in the Application context layer, overriding the standard library binding:

```csharp
public class MyUnity3DApplicationContextInitializer: IApplicationContextInitializer {
	public void Initialize(IDIContext context) {
		context.m().Bind<ILogMessageWriter>(() => new UnityLogMessageWriter());
	}
}
```

Notice, that we don't need to bind *ILogger* again. We just specified: in this application I want to use my own *UnityLogMessageWriter* to write Log messages, but I still want to use whatever logic the dll provides for the *ILogger*. We use *IApplicationContextInitializer*, which is another standard MinDI interface, specifying that this context initializer will populate the application layer. Application layer has the global context layer as the parent reference.  

What now happens if we write 
```csharp
[Injection] public ILogger logger {get; set;}
```
in one of our Unity3D application classes. 

1. First, MinDI will try to find logger in the Application layer context. As it's not found there, it will go by the *parent* reference in the context and will try to find logger in the parent context.
2. As our parent context is the Global context, provided by the dll, the ILogger binding is found there, and is resolved as *new Logger()* instance.
3. Right after creating new Logger, MinDI will start injecting its dependencies, starting from the *resolution context*, i.e. the context we started to resolve this instance. It means it starts to inject the dependencies from the Application layer context. 
4. The ILogMessageWriter dependency exists in the Application context, thus it's resolved from there as *new UnityLogMessageWriter*. Now we use Logger instance, but substituted its dependency on *ILogMessageWriter* with our own instance. If we didn't bind our own *ILogMessageWriter* on the Application level, MinDI would still look into parent context, and eventually would find the resolution to *new ConsoleLogMessageWriter* as defined in the Global layer.  

With the power of the layered context, you can override some of the dependencies when it's needed. On the inferior layers, the dependencies can be overriden for the parts of the application, like for some dynamically spawned objects, using the same way. Our application becomes context-oriented, where the logic depends on the context we are in at the moment. 


## Controlling lifetime

There is 2 principal types of objects lifetime in the context: **concrete** and **abstract**. Speaking in the programming terms, concrete means a *singletone* - an object that exists as the same concrete instance for anybody who depends on it. And the abstract is a *multiple* binding - any class, that depends on this object, will have a new instance created automatically. The *concrete* or *singletone* type of lifetime should not be confused with the *Singletone* anti-pattern. The common between them, is that we provide a way for an object to exist only as one instance. In the concrete lifetime binding there doesn't exist any global singletones though. Any singletone binding is only global within the context layer, it's defined. 

On the human mind analogy level, the singletone binding matches the article "the". It means a concrete one entity, we are talking about in this context. That's why in the terms of the context it's called "concrete". The multiple binding matches the article "a". It means any entity of this type, not existing known instance yet. The concrete and abstract bidnings in the context match the singletone and multiple lifetime in the DI/IoC patterns and is indeed one of the fundamental principles of the programming logic and the human logic in general.

This is an example of concrete and abstract bindings:

```csharp
public class LoggerLibraryGlobalContextInitializer: IGlobalContextInitializer {
	public void Initialize(IDIContext context) {
		context.s().Bind<IApple>(() => new Apple());
		context.m().Bind<IOrange>(() => new Orange());
	}
}
```

Every class that depends on *IApple* will use the same instance of the Apple. Every class that depends on *IOrange* will have it's own, new instance of the Orange.
As we see, **.s()** and **.m()** methods are the special lifetime resolvers to define it. In MinDI you can extend the framework with your own liftime resolvers, so you can have more different liftime types, than just standard ingletone and multiple. This is done when you need some custom behaviour. For example, in ASP.NET MinDI uses *.ses()* lifetime to define a singletone existing in the user session. In Unity3D *.mbm()* and *.mbs()* lifetimes are used to define the MonoBehaviours. 

### Lazy construction
By default the singletones use *lazy construction*. It means that the singletone instance will be resolved as soon as the first class that depends on it is created. However, you can also use *instant construction*. Like this you construct your class immediatly in the context intializer. That can be usefull for huge objects, that you want to create during the initialization of the application. However it's even more often used for simple objects like enums or data types, that don't have external dependencies. The limitation with the *instant construction* is obvious: no dependencies can be resolved for such object, as they are created at the moment where the context is not yet built. However, there is another way of early instantiation, that is performed in the entry point of the application, but after the context is initialized. The instance construction is only recommended for the simple objects, like enums, etc.

Here is an example of the instant construction:

```csharp
public class LoggerLibraryGlobalContextInitializer: IGlobalContextInitializer {
	public void Initialize(IDIContext context) {
		context.s().BindInstance<IApple>(new Apple());
	}
}
```

The instance of the *IApple* is immediately created here. Obviously, **BindInstance** method is only available for the *.s()* lifetime qualifier. 

### Subjective dependencies. Rebinding.

In MinDI, the dependencies are resolved starting from the context on which the object is created. The concrete(singletone) is always created on the same context it is defined. The abstract(multiple) object is always created on the same context it's called from. That implements a subjectivity principle: the abstract objects dependencies are always **subjective**, i.e. they take the dependencies from the context, they are requested from. Thus, requesting the same abstract object from different context, it will have different dependencies. Doing the same with a concrete object, it will have the same dependencies, injected from the context, it's defined in. That principle is important to understand. Let's see the following example.

In the global layer we have: 

```csharp
public class LoggerLibraryGlobalContextInitializer: IGlobalContextInitializer {
	public void Initialize(IDIContext context) {
		context.s().Bind<IAnotherLogger>(() => new SingletoneLoger());
		context.m().Bind<ILogger>(() => new Logger());
		context.m().Bind<ILogMessageWriter>(() => new ConsoleLogMessageWriter());
	}
}

...

internal class Logger: ContextObject, ILogger {
	[Injection] public ILogMessageWriter writer {get; set;}
	...
}

internal class SingletoneLogger: ContextObject, ILogger {
	[Injection] public ILogMessageWriter writer {get; set;}
	...
}

```

In the application context that is the next layer, we define another implementaton of *ILogMessageWriter*. 

```csharp
public class AppContextInitializer: IApplicationContextInitializer {
	public void Initialize(IDIContext context) {
		context.m().Bind<ILogMessageWriter>(() => new MyLogMessageWriter());
	}
}
```

Now let's see what dependencies the objects will have if our *context* is on the application level.

```csharp
var log1 = context.Resolve<ILogger>();
var log2 = context.Resolve<IAnotherLogger>();
```

Here the *log1* will depend on *MyLogMessageWriter*, as it picks the dependencies from the context we requested it from (and upper by the chain of prototype), even though it's defined in the parent context. But the log2 will have the dependency to the *ConsoleLogMessageWriter*! As the singletone is created on the same context, it's defined and picks the dependencies from this context (and upper by the prototype chain). That means that the first initiator of the sigletone instantiation will not be able to inject dependencies from its own context, as this singletone can be later accessed from different contexts as well.

So, what if I want to have a singletone logger in my application, that uses my version of the *ILogMessageWriter*? For this there is another important context feature called *rebinding*. Let's do the following:


```csharp
public class AppContextInitializer: IApplicationContextInitializer {
	public void Initialize(IDIContext context) {
		context.m().Bind<ILogMessageWriter>(() => new MyLogMessageWriter());
		context.s().Rebind<IAnotherLogger>();
	}
}
```

Notice, that we don't specify any lambda-factory in the Rebind construction. That's because we tell MinDI to look in the parent contexts for the definition of ILogger and bind the same, just on this context with the specified lifetime. 

Now if I write 
```csharp
var log2 = context.Resolve<IAnotherLogger>();
```

The resolved logger will have the dependency on *MyLogMessageWriter*. The singletone is redefined on this context, and thus gets the dependencies from the same context it's defined, as already explained above.

The rebinding is very powerfull feature, as it allows to change the lifetime of the objects. The standard approach is that the object is defined as abstract (multiple) in the class library context, as the class library should not impose the lifetime of the objects, and should let the user decide. The user can in the application context rebind all the necessary interfaces as singletone, thus decided which objects will be singletones in this application. The rebinding still doesn't need to know anything about the concrete implementation of the interface, allowing the library context to decide it.

The same principle is usefull when we have multi-layered context within the application, like some of the objects can be rebound with different lifetimes within some custom context.

## Named and default dependencies

One interface can be defined multiple times in the context. To do this, there must be given the name to the binding:
```csharp
public class AppContextInitializer: IApplicationContextInitializer {
	public void Initialize(IDIContext context) {
		context.m().Bind<ILogger>(() => new Logger1(), BindingName.For("logger1"), makeDefault: true);
		context.m().Bind<ILogger>(() => new Logger2(), BindingName.For("logger2"));
	}
}
```

This allows to resolve the different implementations of ILogger by providing different names:

```csharp
var log = context.Resolve<ILogger>(BindingName.For("logger2"));
```

If no name is passed the default binding will be used (specified as makeDefault boolean).

This way we can have many configurable implementations that can be switched in the runtime. The binding name can be any object (ToString is used). The binding names can also be combined using special feature of BindingName helper.

In the user level we don't have access to the context, so to dynamically resolve the binding name, a **dynamic injection** is used:

```csharp
[Injection] public IDynamicInjection<ILogger> loggerInjection { get; set; }
...
void MyMethid(string loggerType) {
	var logger = loggerInjection.Resolve(BindingName.For(loggerType)); 
	...
}
```

In this example we use the loggerType string to dynamically obtain the necessary ILogger implementation in the runtime.

## What's next

This is the end of this overview. Feel free to post any questions. The documentation and tutorials on MinDI will be added later.
For now you can play with MinDI on GitHub here: 
https://github.com/pleasenophp/mindi

You also have access to the following demo projects:
- https://github.com/pleasenophp/mindi-demo
- https://github.com/pleasenophp/mindi-unity-demo


<!--
#### Possible next articles:


1) MinDI overview (platforms, features, installation, etc) + add references.
2) Turorial: hello world, using of basic features and generics, constructions
3) Unity tutorial: hello world, switching between scenes, load scenes as objects

-->
