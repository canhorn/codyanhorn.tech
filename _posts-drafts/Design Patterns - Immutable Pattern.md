---
layout: post
author: Cody Merritt Anhorn
title: Design Patterns - Immutable Pattern
---

The Immutable Pattern is very helpful in understanding when and where an object will be updated, since your not able to change the state of an instances. When the class is structured in a way that the only way to change state is to instantiate a new object through the constructor or builders.

## Usage 

My personal usage of Immutable objects are with the state objects I set on my entities. By making the state object immutable it helps me to keep the object pure and only updated in specific ways. 

Keep track of when an object can be updated, can also be a con of the Immutable pattern. Since the object can only be updated in certain scenarios it can add a bit more ceremony to just change a single property on the instance, and making sure it is updated everywhere.

### Code

~~~ csharp
// This object is structured in a way that the internal state cannot be changed externally.
// With a business logic method that will take the state and change it,
//  then return an updated instances, all without mutating the current instance.
public class LevelState
{
    private readonly int _level = 1;
    public int Level => _level;

    public LevelState(
        int level
    )
    {
        _level = level;
    }

    // Some business logic specific to the LevelState.
    // Will increment the current level and return a new LevelState.
    // This helps to keep this object immutable by not mutating this objects state,
    //  but returning a new object with the NEW expected state.
    public LevelState LevelUp()
    {
        return new LevelState(
            _level + 1
        );
    }
}
~~~
