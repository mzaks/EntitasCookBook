# Collector
_If you did not read the Group chapter, please go read it before you continue here._

A collector is a class which observers a group. Here is an example from MatchOne how you can create a collector:

```csharp
context.CreateCollector(GameMatcher.GameBoardElement.Removed());
```

In this example we define that we want to collect all entities which got `GameBoardElement` component removed. Internally a collector will ask for a group of entities which contain `GameBoardElement` components. It will subscribe it self to group events and keep a list of references to entities which will leave the group, as we were interested in `Removed` event. There are follwoing three events that we can be interested in:
- Added
- Removed
- AddedOrRemoved

Also important to notice, when an entity got collected as removed from a group. It will still stay collected even if we add a `GameBoardElement` component to it again and there for it will be added to the group again. This is why reactive systems has to implement `Filter` method (more on it in reactive systems chapter).

A collector can also be created with an array of groups and events. Meaning that we can observe multiple groups and keep a joined list of changed entites.

A collector can be activated and deactivated, so that we can stop and resume the observing of the group. We can iterate over collected entities and clear them out.

Collector is what powers reactive systems. You probably will not use it stand alone, but it is still a very important ingredient for Entitas.
