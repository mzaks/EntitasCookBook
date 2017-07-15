# Component
Component is the simplest ingredient in ECS. It is an atomic representation of data. It can be empty, have one or many properties, or even be marked as unique. In following I will explain the differences with regards to Entitas-CSharp.

## The simplest Component - Flag Component

I will take as examples, components defined in [Match-One](https://github.com/sschmid/Match-One) example project.

```csharp
using Entitas;

public sealed class MovableComponent : IComponent {
}
```

As you can see a component is a class which implements an `IComponent` interface. It does not have any properties so it is a _flag component_. Flag components are defined to flag entities. In this case we say that something is movable. So if we will have an entity at ahdn we can ask `entity.isMovable` and get `true` or `false` back. We also can ask for all entities which have `MovableComponent`, but this topic I would rather discuss later.

## Data Component
Data component can have multiple properties which can store pure data:

```csharp
using Entitas;

public sealed class PositionComponent : IComponent {
    public int x;
    public int y;
}
```

In Entitas-CSharp we can add a position to an entity with following statement:
```csharp
entity.AddPosition(new PositionComponent(1, 2));
```

There are method to check if an entity `Has` a component. We can also `Get`, `Replace` and `Remove` components. Every entity can have only one type of a component set. This is why we have `Replace` methods. But we can combine all the different types of components in a single entity. This is why it is better to slice your components as thin as posible. This gives you big benefits in terms of feature _improvisation_.

## Reference Component
A reference component is technically equal to the _data component_ the difference is rather logical.

```csharp
using Entitas;
using UnityEngine;

public sealed class ViewComponent : IComponent {
    public GameObject gameObject;
}
```

Technically speaking a reference component is also just a component with multiple properties, but those properties do not represent data, they reference to a complex object. This has a rather profound implication. Those components are harder to serialise. They point to some objects created at runtime, there for it is not usefull to persist those pointer as is. We will have a deep dive in to _reference components_ in recepies section.

## Action Component
This is again just a derivate of a data component. But in this case property is a function/action.

```csharp
using Entitas;
using System;

public sealed class DelegateComponent : IComponent {
    public Action action;
}
```

In this case we can store a function/delegate/action inside of a component and there for attach it to an entity. This is a valid use of components, but it does more harm than good as we will discuss in recepies section.

# Unique Component
In every application there are many cases where you would like to have only one instance of something. This idea manifested itself in the well knowen and often hated _singleton pattern_. In Entitas we have something similar but, better.

Every type of components we discussed previously can be defined as unique component.

```csharp
using Entitas;
using Entitas.CodeGeneration.Attributes;

[Unique]
public sealed class GameBoardComponent : IComponent {
    public int columns;
    public int rows;
}
```

For this we just has to annotate the class as unique.

The framework will make sure that only one instance of a unique component can be present in your context (see context chapter). This is why in Entitas-CSharp we can get an instance of unique component with follwoing expression - `context.gameBoard`.

Now how is it better than _singleton pattern_? It is better due to the fact that we separate state from behaviour. The component can also be replaced and removed. So it breaks the idiom of the _singleton pattern_ where an object is unique and persistant throughout application life cycle. A unique component is more af a gloabel variable than a singleton.