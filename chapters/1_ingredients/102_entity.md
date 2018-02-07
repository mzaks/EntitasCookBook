# Entity

An entity is just a bag of components. We can add a component to an entity, we can get a component from an entity and we can remove a component from an entity. In Entitas-CSharp there is an internal/general way of doing those operations:
```csharp
entity.AddComponent(index, component);
entity.GetComponent(index);
entity.RemoveComponent(index);
```

We have to use an index, becuase the _bag_ is implemented as an array of `IComponent`s. In Entitas-CSharp we chose to use an array for performance reasons. However there are different implementations which chose to use a hash map, making component type the key of the map and component instance the value.
Before you get too upset about the inconvenient API, let me show you how we add, get and remove components in Entitas-CSharp in practice.

Say we have a `PositionComponent`. 
```csharp
public sealed class PositionComponent : IComponent {
    public IntVector2 value;
}
```

Then the API looks as following:

```csharp
entity.AddPosition(new IntVector2(x, y));
entity.position;
entity.RemovePosition();
```

We get this nice API thanks to the code generation tools we implemented for Entitas-CSharp. You can find more on this topic in Code Generation chapter in Appliances section.

## Entity creation

An entity should always be a part of a context. This is why we are not able to instantiate an entity directly, but have to call `context.CreateEntity()`. Context is a managing data structure which monitors entities life cycle. You can find more details about context in Context chapter.

While an entity can be created and destroyed, it is important to know that in Entitas-CSharp destroyed entities are not really destroyed, but object pooled in the context. This is a performance optimisation to avoid garbage collection. The side effect of this fact is that users have to be careful if they keep a reference to an entity in there own code.

When an entity is destroyed it will be put into a temporary pool and reused if it's reference count is back at `0`. 

### What is a reference count?

Reference count is an internal mechanism which makes sure that an entity is not reused before it is not referenced any more. In order to track references an entity has two methods:

`public void Retain(object owner)`

`public void Release(object owner)`

When you keep a reference to an entity, you have to call `entity.Retain(this);` and when it's time to drop the reference it is important to call `entity.Release(this);`. Those calls increase and decrease the reference count. All internal classes of Entitas-CSharp are respecting this mechanism and so should your code. If you don't call `Retain` while keeping a reference to an entity, you might end up holding a reference to an entity which was destroyed and reborn as something else. If you forget to call `Release` on an entity which you retained, it will stay in the object pool forever, making your memory consumption grow over time.

## Entity observation

An entity has multiple events which users can subscribe to, in order to have introspection into entity life cycle.
Here is a list of all the events entity has in the current imlplementation of Entitas-CSharp:

- OnComponentAdded
- OnComponentRemoved
- OnComponentReplaced
- OnEntityReleased
- OnDestroyEntity

Those events are the same events context uses to monitor entity. They are exposed for the external use as well, however I would not recommend to use them directly. In a typical use case you rather want to have a group, collector or a reactive system (described in the respective chapters). However it is good to know that those facilities are present and it could be important to use specifically if you are implementing some tooling.
