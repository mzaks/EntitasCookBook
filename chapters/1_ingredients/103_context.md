# Context
Context is a managing data structure for entities. An entity can not be created stand alone, it has to be created through `context.CreateEntity()`. This way a context can manage the life cycle of all entities we create. It also is the first observer which get notified when we manipulate an entity (see Entity observation section in Entity chapter).

## Entity object pool
In order to avoid garbage collection, a context in Entitas-CSharp has an internal object pool. It contains destroyed entities, which will be used when a user creates a new entity. This way memory on the heap gets recycled. An entity can only be recycled, when we can be sure that no one holds a reference to this entity any more. This is why Entitas-CSharp has an internal reference count mechanism. If you use only stock Entitas and do not hold any references to entites your self, you don't have to think about it. The internal classes already taking care of all reference counting for you. If however you want to create a component like:

```csharp
class Neighbour: IComponent {
    public IEntity reference;
}
```

Or have a _MonoBehaviour_ which references an Entity:

```csharp
class EntityLink : MonoBehaviour {
    IEntity _entity;
}
```
Than you would need to call `_entity.Retain(this);` when you store the reference. And you should not forget to call `_entity.Release(this);` when you are not interested in the entity any more, or the object storing the reference gets destroyed. If you forget to call `Release` a destroyed entity will be kept around and never reused. Effectively it leads to a memory leak, which will be easy to observe in Entitas Visual Debugger. In case you forget to call `Retain`, you might end up with a reincarnated version of an entity. This will lead to very strange behaviour which is very hard to debug.

BTW we discourage components which have references to another entity in favour of an entity index (see Index chapter). And `EntityLink` is now part of `Entitas.Unity` addons, so no need to worry, if you just need to reference an entity on a _GameObject_. We got your back.

## Multiple context types
If we would compare a typical relational (table based) database with Entitas, we could draw following association. A component is a column, an entity is a row and context is a table itself. Now in relational databases a table is defined by a schema. In Entitas it is based on classes which implement `IComponent`. This implies that when we define more component classes, our table becomes broader. Dependent on implementation detail, it can have an implication on memory consumption. In case of Entitas-CSharp it actually does have an implication on memory consumption, as an entity is backed by an array of `IComponent`s.

In order to tackle the growing table size, we can just introduce another table.
Here is a snippet from Entitas-Csharp Wiki:

```csharp
using Entitas;
using Entitas.CodeGenerator;

[Game, UI]
public class SceneComponent : IComponent
{
    public Scene Value;
}

[Game]
public class Bullet
{
    // Since it doesn't derive from 'IComponent'
    // it will be generated as 'BulletComponent'
}

[Meta]
public struct EditorOnlyVisual
{
    public bool ShowInMode;

    public EditorOnlyVisual(bool show) {
        this.ShowInMode = show;
    }
}
```

The annotations above component class declarations tell the code generator which context types we want to have. In this particular example we have a `Game`, `Meta` and `UI` context. As you can see with `SceneComponent`, one component can be part of multiple contexts. Meaning - a table `Game` and table `UI` can both have column `Scene`, if we want to project it again onto realtional database mental model.

### How many context types should I have?
This really depends on your use case. If you have a fairly small/simple game, you can go with just one context. It is much simpler this way. You just have to keep in mind than an entity is backed by an array of `Icomponent`s meaning that it is an array of pointers and a pointer is 8bytes big on an 64bit architecture. So if you have 50 components, every entity will be atleast 400bytes big. If you have 100 entites in your game, they take up 40KB. Now it is up to you to decide if 40KB is alot or not. In case you have hundreds of components and thausends of entites, it would be better to start slicing.

Sometimes it is also benefitial to slice components into different contexts just for organisational purposes. You probably have components which are needed only in core game context and some which are only relevant for meta game. If there is definetly no overlap, meaning there will be no entity which will need to store component `A` and component `Z` than it is better to puth them in different _"tables"_.

## Context observation
Same as with entity a context can be observed for changes. And this is also what we use internally for groups (described in its own chapter) and visual debugger.
If you want to write some tooling for Entitas e.g. custom Logging or profiling you can use follwoing events:

- OnEntityCreated
- OnEntityWillBeDestroyed
- OnEntityDestroyed
- OnGroupCreated