# Index

When we want to get all entities that have a position component, we create a group and iterate over it. However, what about cases, where we want to get entities on a certain position. We could iterate over all entites which have a position and collect only those which have desired position. Or we could use an Index.

In order to create an index we have to annotate component value as following:

```csharp
using Entitas;
using Entitas.CodeGeneration.Attributes;

[Game]
public sealed class PositionComponent : IComponent {

    [EntityIndex]
    public IntVector2 value;
}
```

The `EntityIndex` annotation will tell the code generator to create API on context so that user will be able to get entities by given `IntVector2` value.

Here is a snippet from `ProcessInputSystem` in MatchOne:

```csharp
foreach (var e in _contexts.game.GetEntitiesWithPosition(
                    new IntVector2(input.x, input.y)
                  ).Where(e => e.isInteractive)) {
    e.isDestroyed = true;
}
```

In this snippet we ask context to give us all entities on position, where the "input" was effected and we filter out entites which are not interactive.

Internally an index is a group observer. It is created on context initialisation, subscribing to group events from the beginning. When we start to create entities and add components to them, they will start entering groups and the index will be notified that an entity was added with following component. We can use the value of the component as a key in a HashMap, where the value is the entity itself. This way we are building up an index. When we replace or remove component. The index is notified by the group as well. It gets the previous component so it can remove the entry from the HashMap and if we replaced it with another component, it will receive the added event again with the new value.

In Entitas-CSharp we have two types of built in indexes. `EntityIndex` and `PrimaryEntityIndex`. An `EntityIndex` is backed by a HashMap which stores a set of entities as value. Meaning - you could have multiple entities on the same position. `PrimaryEntityIndex` makes sure that every key is associated with only one entity. This is very good if you have an `Id` component and you want to look up entities by this `Id`. This is also what we recommend, when you need to store a reference from one entity to another (more on it in ingredience chapter).

As mentioned in previous paragraph, Entitas-CSharp implements only two simple entity indexing strategies. You might need a more complex one, where you would like to get entities in a range, or have a more complex index key. In this case please have a look at `AbstractEntityIndex` class. Armed with the knowledge provided in this book, it should be fairly easy to understand the implementation and write your own custom index.
