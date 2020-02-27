---
layout: post
author: Cody Merritt Anhorn
title: Design Patterns - Composite Pattern
---

I personally use the composite pattern in my game design quite heavily, to start off the composite pattern can be seen a taking a single entity and attaching functionality to it by adding other entities to it.

## What is a Composite Entity

To quote [Lexico](https://www.lexico.com/definition/composite)

> A thing made up of several parts or elements.

To break that down we add new functionality to an already existing object allowing for grouping of related components together.

## Example

In this example I will be going over what I use in my Game Server to construct, manage, and expand functionality on the entities managed by the server.

My game server its self has two main concepts the composite pattern is used, one to hold module state and the other to hold general properties. A module state relates to the persistent state of the entity, like what model they are using, what level details they have, or what the state of their skill tree is. 

Using the composite pattern I can attach the different states to the entities and they all can have different, independent managed states, or have a completely different collection of states. Below I will show the gist of the concept used in the confines of a game entity and state properties.

### Code

~~~ csharp
using System;
using System.Linq;
using System.Collections.Generic;

namespace EventHorizon.Patterns.Composite
{
    class Program
    {
        static void Main(
            string[] args
        )
        {
            // Player Entity
            var player = new Entity(
                "Player"
            );
            // Add a Level State property to the Player
            player.SetProperty(
                "Level",
                new LevelState(
                    99
                )
            );
            // Add an Account State to the Player
            // This is specific to the Player Entity 
            //  and other entities will not need this property.
            player.SetProperty(
                "Account",
                new AccountState(
                    Guid.NewGuid().ToString()
                )
            );
            // Guard Entity
            var guard = new Entity(
                "Guard"
            );
            // Add a Level State to the Guard Entity
            guard.SetProperty(
                "Level",
                new LevelState(
                    5
                )
            );
            // Monster Entity
            var monster = new Entity(
                "Monster"
            );
            // Add a Level State to the Monster Entity
            monster.SetProperty(
                "Level",
                new LevelState(
                    1
                )
            );

            // Print out the details for the entities
            PrintDetails(
                player
            );
            PrintDetails(
                guard
            );
            PrintDetails(
                monster
            );

            // Run some business logic on the Player
            RunLevelUp(
                player
            );

            // Print out the details for the entities,
            //  after running some business logic.
            PrintDetails(
                player
            );
            PrintDetails(
                guard
            );
            PrintDetails(
                monster
            );
        }

        // Take in an entity and if they contain a LevelState property
        //  run the LevelUp method on the LevelState
        static void RunLevelUp(
            Entity entity
        )
        {
            var property = entity.GetProperty<LevelState>(
                "Level"
            );
            if (property != null)
            {
                entity.SetProperty(
                    "Level",
                    property.LevelUp()
                );
            }
        }

        // Just loop through all the properties on the entity
        //  calling the Details method to display internal details
        //  about the state.
        static void PrintDetails(
            Entity entity
        )
        {
            System.Console.WriteLine(
                "Entity ({0}) Details: ",
                entity.Name
            );
            foreach (var component in entity.Properties)
            {
                System.Console.WriteLine(
                    "Property : {0}",
                    component.Details()
                );
            }
            System.Console.WriteLine("----");
        }
    }

    // This is the Entity class.
    // This class follows the Composite Pattern, by having a Collection of
    //  properties, the class contains get/set methods to manage its internal state.
    public class Entity
    {
        private readonly string _name;
        private IDictionary<string, Property> _properties = new Dictionary<string, Property>();
        public string Name => _name;
        public IList<Property> Properties => _properties.Values.ToList().AsReadOnly();

        public Entity(
            string name
        )
        {
            _name = name;
        }

        // This takes in a Key and Property instance.
        // Will override any already existing property.
        public void SetProperty(
            string key,
            Property property
        )
        {
            this._properties[key] = property;
        }

        // This will return a property casted to the wanted type.
        // If it is not found it will return a default typed property.
        public T GetProperty<T>(
            string key
        )
        {
            if (_properties.TryGetValue(
                key,
                out var property
            ))
            {
                return (T)property;
            }
            return default(T);
        }
    }

    // A simple interface that can be used to create concrete properties.
    public interface Property
    {
        string Details();
    }

    // A Level abstraction property, to contain an entities level details.
    public class LevelState : Property
    {
        private readonly int _level = 1;
        public int Level => _level;

        public LevelState(
            int level
        )
        {
            _level = level;
        }

        public string Details()
        {
            return $"Level is {_level}";
        }

        // Some business logic specific to the LevelState.
        // Will increment the current level and return a new LevelState.
        public LevelState LevelUp()
        {
            return new LevelState(
                _level + 1
            );
        }
    }

    // An account abstraction property, to hold the entities Account Id.
    public class AccountState : Property
    {
        private readonly string _accountId = string.Empty;
        public string AccountId => _accountId;

        public AccountState(
            string accountId
        )
        {
            _accountId = accountId;
        }

        public string Details()
        {
            return $"AccountId is {_accountId}";
        }
    }
}

~~~
