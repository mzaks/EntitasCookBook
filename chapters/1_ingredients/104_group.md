# Group
The typical _"Hello World"_ of ECS is the so called "move system". A move system is a system which takes all entites which have position and velocity components and exchanging the position component on this entity, effectivly moving it toward the velocity vector. The __AHA__ moment comes when we realise that it does not matter what kind of other components are on this entity. It can be a person, dog, car, helicopter or a house. If it has a position component and a velocity component, it has to be moved.

### How do we get those entities though?
As described in context chapter, a context manages all the entites so we could ask context for all entities and iterate through all of them, collecting those who has position and velocity components. This would be a very naive implementation. What we have in Entitas for this case is a so called __Group__.

```csharp
context.GetGroup(GameMatcher.AllOf(GameMatcher.Position, GameMatcher.Velocity));
```

In the statement above we ask a context to provide us with a group holding entites which have `Position` and `Velocity` components. A group is a collection of entites which is always up to date. Meaning if you remove a position from an entity it will immediately leave this group. And if you add a position and velocity components to an entity it will directly enter the group.

You can ask for groups without any hesitation, because they are internally reused. A context keeps an internal list of all groups you asked, so if you ask again for a group with same matcher it will just give you a reference to already existing one. Speaking of matchers...

## Matcher 
A matcher is a way how we can describe what kind of entites we are interested in. It is our small query language if you will. `GameMatcher` means that we have a `Game` context (see Multiple context types section in Context chapter) and we can access all component types associated to this context. If we would write `context.GetGroup(GameMatcher.Position);` We would get a group of entites which have `Position` component. In order to define more complex groups, we can use `AllOf`, `AnyOf` and `NoneOf` methods. `AllOf` means that all the listed components has to be present on the entity in order for this entity to become part of the group. `AnyOf` means that one of the listed component has to be present. And in case of `NoneOf` we don't want the listed components to be present. `NoneOf` is not a stand alone description, meaning that you will not be able to write `context.GetGroup(GameMatcher.NoneOf(GameMatcher.Position));` It is prohibited because it creates a very large set. `NoneOf` can be used only in combination with `AllOf` or `AnyOf`.

```csharp
context.GetGroup(GameMatcher.AllOf(GameMatcher.Position, GameMatcher.Velocity).NoneOf(GameMatcher.NotMovable));
```

This way we can say that we need a group of entities which have `Postion` and `Velocity` components but does not have `NotMovable` component.

`AllOf` and `AnyOf` can also be combined: `context.GetGroup(Matcher.AllOf(Matcher.A, Matcher.B).AnyOf(Matcher.C, Matcher.D).NoneOf(Matcher.E))`

A matcher definiton can also start with `AnyOf`: `context.GetGroup(Matcher.AnyOf(Matcher.C, Matcher.D).NoneOf(Matcher.E))`

# Group observation
As I mentioned before a group is always up to date, so it provides a great benefit if we can observe a group and get notified when an entity was added or removed from it. Even more importantly is to understand than, when we replace a component on an entity, old component will be removed and new component will be added. This means that the entity will leave a group and than reenter it with a new value. This is what provides us with foundation for reactive programming.

Internally in Entitas-CSharp we don't really remove and add components. The generated code asks user for new values, fires the events as if we would remove the component with old values, sets new values in the component and fires and event as if a new component was added. This way we avoid memory allocation and simulate a feeling of working with immutable components.

A group has follwoing events you can subscribe to:
- OnEntityAdded
- OnEntityRemoved
- OnEntityUpdated

Other ingredients like Collector, Index and Reactive system are using the same events. So, for day to day work, you probably can use those. But if you want to build something custom, you might want to have a look at implementation details.
