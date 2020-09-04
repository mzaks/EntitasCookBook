# "Hello world"

"Hello world" is a typical example, which is used to illustrate the key benefits of _technology at hand_. For programming languages, people use a small programm which outputs __Hello World__ to the console. In case of an ECS most implementations I saw, start with a `MoveSystem`.

## Move System

Move system is a system which moves entities. In order to move something, this something needs to have a position and a velocity:

```csharp
using Entitas;

[Game]
public sealed class PositionComponent : IComponent {
    public int x;
    public int y;
}

[Game]
public sealed class VelocityComponent : IComponent {
    public int x;
    public int y;
}
```

If something has a position and a velocity, we can find it through movable group. Than we can iterate over all movables and replace position of a movable with it's previous position plus the velocity.

```csharp
using Entitas;

public sealed class MoveSystem : IExecuteSystem {

    Group movableGroup;

    public MoveSystem(Contexts contexts) {
        var matcher = Matcher.AllOf(GameMatcher.Velocity, GameMatcher.Position);
        movableGroup = contexts.game.GetGroup(matcher);
    }

    public void Execute() {
        foreach(var e in movableGroup) {
            e.ReplacePosition(e.postion.x + e.velocity.x, e.postion.y + e.velocity.y);
        }
    }
}
```

This is how we can move things with Entitas. It is a very simple example, but it illustrates the strength of an ECS. This one system can find every relevant entity and move it. We don't care what kind of other component are attached to this entity, it can be a car a human a dog a house, we don't care. We just defined that if something have a position and a velocity, it will be moved.

## Decelerate System

With the given move system we will keep moving things at a constant speed until we stop the game / application. However in most cases we would like to decelerate things. Let's do it in a separate system.

```csharp
using Entitas;

public sealed class DecelerateSystem : IExecuteSystem {

    Group velocityGroup;

    public DecelerateSystem(Contexts contexts) {
        velocityGroup = contexts.game.GetGroup(GameMatcher.Velocity);
    }

    public void Execute() {
        foreach(var e in velocityGroup) {
            var velocityX = e.velocity.x / 2;
            var velocityY = e.velocity.y / 2;
            if (velocityX == 0 && velocityY == 0) {
                e.RemoveVelocity();
            } else {
                e.ReplaceVelocity(velocityX, velocityY);
            }
        }
    }
}
```

The implementation above is very naive, it just devides the velocity in half, so the velocity will become zero vector quite rapidly and when it does, we remove it all together. I don't expect you to write something like this in real code, but it is good enough to show one key concept of ECS. It is a good idea to make systems follow [single responsibility principle](https://en.wikipedia.org/wiki/Single_responsibility_principle). Move system is only concerned with _movable_ entites. Deceleration system is concerned with reducing the velocity. We could even say that current implementation of deceleration system has two resposibilities. It reduces velocity and also removes it. We could even go so far to introduce a third - stop system. Which can be a reactive system, removing the velocity when it changes to be a zero vector.

```csharp
using Entitas;

public sealed class StopSystem : ReactiveSystem<GameEntity> {

    protected override ICollector<GameEntity> GetTrigger(IContext<GameEntity> context) {
        return context.CreateCollector(GameMatcher.Velocity);
    }

    protected override bool Filter(GameEntity entity) {
        return entity.hasVelocity;
    }

    protected override void Execute(List<GameEntity> entities) {
        foreach (var e in entities) {
            if (e.velocity.x == 0 && e.velocity.y == 0) {
                e.RemoveVelocity();
            }
        }
    }
}
```

Now it is possible for us to simplify the decelerate system, by removing the velocity check and just replacing current velocity with reduced velocity.

```csharp
using Entitas;

public sealed class DecelerateSystem : IExecuteSystem {

    Group velocityGroup;

    public DecelerateSystem(Contexts contexts) {
        velocityGroup = contexts.game.GetGroup(GameMatcher.Velocity);
    }

    public void Execute() {
        foreach(var e in velocityGroup) {
            var velocityX = e.velocity.x / 2;
            var velocityY = e.velocity.y / 2;
            e.ReplaceVelocity(velocityX, velocityY);
        }
    }
}
```

## Epiphany

This is all interesting, but I still did not explain my main point. My __main point__ is that systems don't communicate to each other directly. In object oriented programming, functional programming and even procedural programming - objects, functions, or procedures _talk_ to each other directly and syncronously. In ECS systems only query state and change state. They are decoupled from each other. Or atleast they are unaware of each others existence. You might argue that things are coupled through component types, but component types are data driven. We define them in order to reflect the _"world"_ we want to simulate, so __data__ becomes our [API](https://en.wikipedia.org/wiki/Application_programming_interface).

It is a very unique approach in handling things resulting from what I call the first rule of ECS - __Separate state from behaviour__. This is why I think it is worthy of being _Hello World_.
