# Reactive System
Reactive system is a system which will be called only when there are entities we need to process. Internally a reactive system uses a `Collector` instance (see collector chapter for more info). As a user you need to extend an abstract class `ReactiveSystem`. Here is an example of a reactive system from MatchOne:

```csharp
using System.Collections.Generic;
using Entitas;

public sealed class DestroySystem : ReactiveSystem<GameEntity> {

    public DestroySystem(Contexts contexts) : base(contexts.game) {
    }

    protected override ICollector<GameEntity> GetTrigger(IContext<GameEntity> context) {
        return context.CreateCollector(GameMatcher.Destroyed);
    }

    protected override bool Filter(GameEntity entity) {
        return entity.isDestroyed;
    }

    protected override void Execute(List<GameEntity> entities) {
        foreach (var e in entities) {
            e.Destroy();
        }
    }
}
```

The purpose of this system is to destroy entities which are marked with `Destroyed` component. As you can see in the `GetTrigger` method, we return a Collector which monitors a group of `Destroyed` entities. In `context.CreateCollector(GameMatcher.Destroyed)` we did not specify the event on which an entity will be collected, because the default event is `Added`. So when we add a `Destroyed` component to an entity, the entity will be `Added` to `Destroyed` group and there for collected by the collector by the given reactive system.

A reactive system is triggered periodically / on every `Update` same as the execute systems, however the `Execute(List<GameEntity> entities)` method will be executed only if the collector could collect entities between this an previous `Execute` call.

You might wonder what the `Filter` method is all about. As mentioned in Collector chapter when an entity gets collected, it stays collected, even if there were events which logically might feel like reversion of the first event. In our particular example, we say that we collect all entities which were destroyed. So even if something removed the `Destroyed` component from the entity and by this action revived it - the entity still stays collected and will be passed to the `Execute` method. Except we filter it out. In the `Filter` method we can decide if a collected entity should be passed to the `Execute` method. If you don't have any special criteria, you can just return `true`. Otherwise as in this particular example, we can check if the collected entity still has the `Destroyed` component on it.

# Careful with AnyOf based collector
When you create a collector which whatches a group based on `AnyOf` matcher, you probably will get an unexpected result, as when you have components `A` and `B` and you have an `AnyOf(A, B)` group. An entity will enter a group only when one of the components is added, when we add the second component, the entity is still in the group so it is not `Added` and therefore it is not collected. This is however probably not what you want to have. Normally people want to see entities collected when any of the two components are added. In this case what you should do is to setup a collector with two distinct groups and not one `AnyOf` group. Here is an example from MatchOne:

```csharp
using System.Collections.Generic;
using DG.Tweening;
using Entitas;
using Entitas.Unity;
using UnityEngine;

public sealed class RemoveViewSystem : ReactiveSystem<GameEntity> {

    public RemoveViewSystem(Contexts contexts) : base(contexts.game) {
    }

    protected override ICollector<GameEntity> GetTrigger(IContext<GameEntity> context) {
        return context.CreateCollector(
            GameMatcher.Asset.Removed(),
            GameMatcher.Destroyed.Added()
        );
    }

    protected override bool Filter(GameEntity entity) {
        return entity.hasView;
    }

    protected override void Execute(List<GameEntity> entities) {
        foreach (var e in entities) {
            destroyView(e.view);
            e.RemoveView();
        }
    }

    void destroyView(ViewComponent viewComponent) {
        var gameObject = viewComponent.gameObject;
        var spriteRenderer = gameObject.GetComponent<SpriteRenderer>();
        var color = spriteRenderer.color;
        color.a = 0f;
        spriteRenderer.material.DOColor(color, 0.2f);
        gameObject.transform
                  .DOScale(Vector3.one * 1.5f, 0.2f)
                  .OnComplete(() => {
                      gameObject.Unlink();
                      Object.Destroy(gameObject);
                  });
    }
}
```

In this reactive system, we say that we will remove a view, when `Asset` component is removed from an entity, or if `Destroyed` component is added to an entity.

There is however one caveat with this solution. Even though a collector can be setup with multiple groups, all of those groups have to be based on components from the same context. In case you need to have a reactive system which can collect entities from different context types, you need to extend the abstract `MultiReactiveSystem` class. In this class `GetTrigger` method return an array of collectors.
