<!--
.. title: Using real JavaScript with Unity
.. slug: using-real-javascript-with-unity
.. date: 2020-05-19 18:54:56 UTC+02:00
.. tags: unity, javascript, jint, webpack, npm
.. category: Programming
.. link: 
.. description: 
.. type: text
-->

## What

This tutorial will tell you how to use [Jint](https://github.com/sebastienros/jint) engine to write cross-platform games in a real [JavaScript ES6](https://www.w3schools.com/js/js_es6.asp) with [Unity](https://unity.com).
It's not to be confused with UnityScript language, that is .NET js-like syntax, and has little common with JavaScript.

## Why

If you make relatively complex games, like RPG-s, and so on, you probably need a good scripting language, to handle complex game story, NPC-s and object interactions, custscenes, events, and so on. Then, while C# is good for engine logic, it's not designed for scripting. It's simply has too much boilerplate, and too little flexibility for the creative part of the game. You will probably also need a scripting language that is easily understandable by game scripters, and modders, that are not necessary programmers. Many big projects choose [Lua](https://www.lua.org/) for this purpose. Lua is a great language, and has a lot of similarities with JavaScript. You can also make Lua work with Unity. However, here I want to show how to use JavaScript, because it gives the following advantages:

- It has a familiar C-like syntax, as opposite to a bit weird syntax of Lua.
- It has a huge developers community, and npm library with tons of open source packages, including the game development ones, like dialogs, quests, pathfinder, and anything else.
- It has a lot of good established development tools, including unit test libraries, linters, etc.

If you decide to use **JavaScript** in **Unity**, among many other features, you are able:

- Write logic of your game in a multi-paradigm, dynamically typed, easy to read and maintain language with strong concepts, where you can both create a beautiful architecture, and unleash your creativity when coding the game world without loosing focus on technical stuff.
- Make your game scripts logic abstracted from lower level engine logic, also allowing to write automated tests for your story, dialogs and interactions, without even running Unity engine.
- Easily expose your game logic to the community, so fans can create mods and addons.
- Make your game portable to any other engine than Unity, if needed
- Have access to npm library, webpack, babel, and any other mainstream javascript tools and packages.

If you are a professional JavaScript developer, or if you just love JavaScript, but want to make a Unity game, then this tutorial can be especially good for you.

## This tutorial will cover

- [Setting up a simple Unity project with **Jint**.](#1-create-project)
- [Calling Unity **C#** code from **JavaScript**](#2-call-csharp)
- [Loading the script from file](#3-load-js-files)
- Handling **exceptions**
- Setting up **Webpack** and **Babel** to have ES6 support in a fully-functional npm-based project
- **Saving and loading** the game: a simple way to manage state of your **JavaScript** logic.
- Make setTimeout and promises work. Integrating with the Unity coroutines.
- Setting up unit tests for the game logic in **JavaScript**

## Prerequisites

- You have some experience in both JavaScript, C# and Unity
- You have some experience with command line tools and npm

This tutorial will use very simple MonoBehaviour code as example, as its goal is to show how to use Javascript in Unity, not how to create an engine architecture, or a game.

## Let's do it

To run JavaScript engine from .NET we will use [Jint](https://github.com/sebastienros/jint) library. Jint is a Javascript interpreter for .NET which provides full ECMA 5.1 compliance and can run on any .NET platform.

<a name="1-create-project"></a>
### Creating a project and setting up Jint

1) Create a project in Unity

2) Get the **Jint.dll** from NuGet packages. For this do the following:

* The latest stable version at the moment of writing this is *2.11.58*, so download the package from [here](https://www.nuget.org/packages/Jint/2.11.58)
* Rename *jint.2.11.58.nupkg* to *jint.2.11.58.zip* and unpack it
* Take the *Jint.dll* from the folder *lib/netstandard2.0* of the package

3) Make sure your projects uses *.NET Standard 2.0* in *Edit -> Project Settings -> Player*.
This is recommended, as it has smaller size and gives the compatibility with all the platforms Unity supports.

**pic1**

**NOTE**: if you rather want to use .NET 4.x setting than take the *Jint.dll* from the corresponding folder from the package in the previous step.

4) Create a folder *Plugins* in your *Assets* and drag *Jint.dll* there.

**pic2**

5) Let's create a C# MonoBehavior called *JavascriptRunner.cs* on a scene object and call some JavaScript from inside of it:

```csharp
using UnityEngine;
using Jint;
using System;

public class JavascriptRunner : MonoBehaviour
{
    private Engine engine;
    
    // Start is called before the first frame update
    void Start()
    {
      engine = new Engine();
      engine.SetValue("log", new Action<object>(msg => Debug.Log(msg)));

      engine.Execute(@"
        var myVariable = 108;
        log('Hello from Javascript! myVariable = '+myVariable);
      ");
    }
}
```

Here we create new *Engine* object from Jint and call a very simple Hello World code in JS.
Now attach the JavascriptRunner to the *MainCamera* on the scene and press **Play**.

You will see the following output in the console:

**pic3**

Note how we make JavaScript call the Unity *Debug.Log* by proxying the call to *log* function in JavaScript:
```csharp
engine.SetValue("log", new Action<object>(msg => Debug.Log(msg)));
```

This is direct function call, where you can call any C# functions from javascript. There is also other ways to call the C# code, let's see them.

<a name="2-call-csharp"></a>
### Calling Unity C# code from JavasSript

There are several ways to bind C# objects to JavaScript.
As shown above, we can easily bind C# functions to JS. For non-void functions, that need to return value, you can use **Func** delegate. Change the code as following and press Play:

```csharp
void Start()
{
  engine = new Engine();
  engine.SetValue("log", new Action<object>(msg => Debug.Log(msg)));
  engine.SetValue("myFunc", 
    new Func<int, string>(number => "C# can see that you passed: "+number));

  engine.Execute(@"
    var responseFromCsharp = myFunc(108);
    log('Response from C#: '+responseFromCsharp);        
  ");
}
```

Now you can see on the Console:
```bash
Response from C#: C# can see that you passed: 108
```
We created a function that JavaScript can call and get some value from your C# API.

But Jint would not be so powerful if it didn't allow to proxy the whole class from C# to Javascript. That's very handy when you need to give the JS engine access to part of your API.
Let's do it. Modify the code as following and run it:

```csharp
using UnityEngine;
using Jint;
using System;
using Jint.Runtime.Interop;

public class JavascriptRunner : MonoBehaviour
{
    private Engine engine;

    private class GameApi {
      public void ApiMethod1() {
        Debug.Log("Called api method 1");
      }

      public int ApiMethod2() {
        Debug.Log("Called api method 2");
        return 2;
      }
    }

    // Start is called before the first frame update
    void Start()
    {
      engine = new Engine();
      engine.SetValue("log", new Action<object>(msg => Debug.Log(msg)));

      engine.SetValue("GameApi", TypeReference.CreateTypeReference(engine, typeof(GameApi)));

      engine.Execute(@"
        var gameApi = new GameApi();
        gameApi.ApiMethod1();
        var result = gameApi.ApiMethod2();
        log(result);
      ");
    }
}
```

Notice, that we added a class **GameApi** and proxied it to Javascript. You can proxy like this any C# class, or even Enums, that is very handy:
```csharp
engine.SetValue("GameApi", TypeReference.CreateTypeReference(engine, typeof(GameApi)));
engine.SetValue("MyEnum", TypeReference.CreateTypeReference(engine, typeof(MyEnum)));
```

To use it in javascript, we instantiate it using *new* operator:
```js
var gameApi = new GameApi();
```

Other than that, we can also proxy an existing instance of a C# class to exchange data between the Unity and Javascript engine. Let's say we have a *WorldModel* object, that has some data, and we want to proxy it to JavaScript: 

```csharp
using UnityEngine;
using Jint;
using System;

public class JavascriptRunner : MonoBehaviour
{
    private Engine engine;

    private class WorldModel {
      public string PlayerName {get; set; } = "Alice";
      public int NumberOfDonuts { get; set; } = 2;

      public void Msg() {
        Debug.Log("This is a function");
      }
    }

    // Start is called before the first frame update
    void Start()
    {
      engine = new Engine();
      engine.SetValue("log", new Action<object>(msg => Debug.Log(msg)));

      var world = new WorldModel();
      engine.SetValue("world", world);
      Debug.Log($"{world.PlayerName} has {world.NumberOfDonuts} donuts");

      engine.Execute(@"
        log('Javascript can see that '+world.PlayerName+' has '+world.NumberOfDonuts+' donuts');
        world.Msg();
        world.NumberOfDonuts += 3;
      ");

      Debug.Log($"{world.PlayerName} has now {world.NumberOfDonuts} donuts. Thanks, JavaScript, for giving us some");
    }
}
```

Press Play and watch the fun on the Console. Here we have proxied an existing object to JavaScript. You can see that we can both read and write to C# object from the JS side. Like this you can easily expose the shared data to your JS engine.

There are also several other ways of exposing the C# code to JavaScript. You can even expose the whole CLR with all namespaces, even though it's not recommended. You would rather expose only the API that your scripter or modder is supposed to call. But if you need to get more knowledge about interoperability, read the [Jint manual](https://github.com/sebastienros/jint)

<a name="3-load-js-files"></a>
### Loading the scripts from files

Of course we will have our JavaScript code sitting somewhere in files, not hardcoded in C# like in examples above. Let's do this, so we later can start to setup the whole JavaScript project for our game.

































