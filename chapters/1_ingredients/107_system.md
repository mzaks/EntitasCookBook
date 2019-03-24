# System
The main goal of ECS or data oriented design is to separate state form behaviour. System is a place were we define the behaviour. In a system we can write code which creates new state, changes given state or destroys state.

In Entitas-CSharp we have multiple interfaces which we have to implement in order to mark a class as a system. `ISystem` interface is a base interface, which we don't have to implement ourselves. It is just a marked (so called ghost protocol) which is used internally.

If we want to have a system which should be executed periodically, we need to implement `IExecuteSystem`. This interface has only one method `void Execute();`. This is the method where we put the code which should be executed on every tick.

Another type of systems which are executed periodically is `ICleanupSystem`. This one is meant for logic which should be executed after all `IExecuteSystem`s run. As the name suggests, you should put clean up code in its `void Cleanup();` method. As those are just interfaces, we can have one class which implements both those protocols. Some times it totally makes sense from game logic stand point.

### Setup and Teardown

Normally when we start a game we need to create the initial state first. This is why in Entitas-CShapr we have `IInitializeSystem` interface. It has a `void Initialize();` method which should contain your games initialisation logic - basically creating all the entities and other state you need to start playing.

The counterpart of `IInitializeSystem` is `ITearDownSystem`. This one has `void TearDown();` method, where we put code which will be executed before we close the game/Level/Scene (what ever fits your use case).

# Composing systems
Everything I described in this chapter till now, were only interfaces which reflect conventions we use to break apart behaviour code. I saw projects where people used Entitas without systems. They implemented there own Command Pattern. But if you want to follow along with the systems aproach you might want to compose systems together in a certain hierarchy. In order to do this we provide a `Systems` class, which implements the `IInitializeSystem, IExecuteSystem, ICleanupSystem, ITearDownSystem` interfaces. We can `Add` a system to an instance of the `Systems` class. So that when we call an `Execute(), Cleanup(), Initialize(), TearDown()` method on this instance it will call those methods on the added systems. `Systems` class is a typical parent node in sense of [Composite pattern](https://en.wikipedia.org/wiki/Composite_pattern). 

When we look at the MatchOne example we can see that we don't use `Systems` class directly:

```csharp
public class MatchOneSystems : Feature {

    public MatchOneSystems(Contexts contexts) {

        // Input
        Add(new InputSystems(contexts));

        // Update
        Add(new GameBoardSystems(contexts));
        Add(new GameStateSystems(contexts));

        // Render
        Add(new ViewSystems(contexts));

        // Destroy
        Add(new DestroySystem(contexts));
    }
}
```

Here we extend `Feature` class. Feature class is a generated class which extends either `Systems` class or `DebugSystems` class, dependent on if we want to run with visual debugging enabled. Visual debugging consumes lots of resources and should not be enabled when you do a production build, or run your game mobile device. This is why `Feature` class is generated to make our real code simpler.

Another thing that you might noticed from the snippet above, is that we pass in a `Contexts` class. `Contexts` class is another convinience class generated in Entitas-CSharp were we can reference different context instances. The generated code contains getters for an instance of every context type (read more about code generator in appliances chapters).

# How do we execute the systems
After we implemented the systems and combined them into hierarchies, we need to call the `Execute(), Cleanup(), Initialize(), TearDown()` method somewhere. For this purposes we normally create a `MonoBehaviour` which does it. If you are not using Unity3D you would need to see for yourself where you would like to trigger those methods. I would like to use MatchOne again to show how such `MonoBehaviour` class might look like:

```csharp
using Entitas;
using UnityEngine;

public class GameController : MonoBehaviour {

    Systems _systems;

    void Start() {
        Random.InitState(42);

        var contexts = Contexts.sharedInstance;

        _systems = new MatchOneSystems(contexts);

        _systems.Initialize();
    }

    void Update() {
        _systems.Execute();
        _systems.Cleanup();
    }

    void OnDestroy() {
        _systems.TearDown();
    }
}
```

A question which comes up quite frequently is, if the periodical systems should be executed on `FixedUpdate` rather than `Update`. This is generally your personal desicion to make. I schedule the systems normally on `Update`, if in your case it is important to schedule on `FixedUpdate` or even `LateUpdate` it is your decision to make. You could even go bananas and have multiple system hierarchies, where one is executed on `Update` and another on `FixedUpdate`, not sure it is a good idea though.

# How do I implement a typical execute system?
An execute system is run periodically, so what we normaly do is, we set one or multiple groups in system constructor and then in `Execute` we iterate over entities in those groups and change them or create new entities.

Generally speaking, we are pulling data from the context and doing something with it. In Entitas-CSharp there is also another way of dealing with data, you will learn all about it in the next chapter.
