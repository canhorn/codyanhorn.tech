---
layout: post
author: Cody Merritt Anhorn
title: Composite Pattern
---

I personally use the composite pattern in my game design quite heavily, to start off the composite pattern can be seen a taking a single entity and attaching functionality to it by adding other entities to it.

# What is a Composite Entity

To quote [Lexico](https://www.lexico.com/definition/composite)

> A thing made up of several parts or elements.

To break that down we add new functionality to an already existing object allowing for grouping of related components together.

# Example

In this example I will be going over what I use in my Game Server to construct, manage, and expand functionality on the entities managed by the server.

My game server its self has two main concepts the composite pattern is used, one to hold module state and the other to hold general properties. A module state relates to the persistent state of the entity, like what model they are using, what level details they have, or what the state of their skill tree is. 

Using the composite pattern I can attach the different states to the entities and they all can have different, independent managed states, or have a completely different collection of states. Below I will show the gist of the concept used in the confines of a game entity.

TODO: Create Project
TODO: Create Simplified examples for post

-- Entity
-- ModelState
-- LevelState
-- AccountState

~~~ csharp
using System;
using System.Linq;
using System.Collections.Generic;

namespace Project
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
            player.SetProperty(
                "Level",
                new LevelState(
                    99
                )
            );
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
            monster.SetProperty(
                "Level",
                new LevelState(
                    1
                )
            );

            PrintDetails(
                player
            );
            PrintDetails(
                guard
            );
            PrintDetails(
                monster
            );

            RunLevelUp(
                player
            );

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

        public void SetProperty(
            string key,
            Property property
        )
        {
            this._properties[key] = property;
        }

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

    public interface Property
    {
        string Details();
    }

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

        public LevelState LevelUp()
        {
            return new LevelState(
                _level + 1
            );
        }
    }

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
