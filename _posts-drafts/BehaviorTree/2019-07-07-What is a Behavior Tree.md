---
layout: post
author: Cody Merritt Anhorn
title: What is a Behavior Tree?
---

## In the context of a Video Game

It is a graph that dictates the state transitions an Game Agent or Actor will make.

## My usage

In the confines of my personal project, EventHorizon Immortals, I wanted a way to structure an agents behavior in a data driven way. Using JSON I was able to create a tree of different types of behaviors I wanted my agents to follow.
In the example I will be using through this post I will be showing off a simple wander tree.

Most of the implementation of the BT I have used was from the articles from Björn Knafla. Using his articles to validate the way the BT behaves is what it should be.

The first pass of creating the logic required making a test of calling the "kernel". The kernel would be the main processing loop of the BT shape. The shape in the context of the BT tell the logic where the graph starts and the order the nodes will be processed in. When running a BT the interface expects the Shape and the Agent to be passed. The first pass at this did not reuse any type of BT state from Agent, in the first pass of creating the BT I thought this was acceptable.

The processing did work, I was able to get the agent to correctly behave based on the shape of the initial BT state. I will go over the transition to using any prior state the Agent might of been in further into the article, but for now we will go over the implementation for this single pass BT implementation. Since the structure of the BT processing does not change much from this initial pass to the full featured state management, we will be building up from there.

## Who Helped

Going through the articles from Björn, we find a few areas we will need to explain to help understand how a BT functions. The first is what a Node is, we can think of a Node a location on the graph that is currently being processed. Every graph has a Root node, this is the starting location of every call to process a BT. 

Behavior Trees have a few types that we can explain here, I will only be explaining the one used by my platform, you can see the definitions of the others in Björn article. We will first start off with the explanation of the Traversal nodes, traversal nodes are nodes that lead to sub nodes be that more traversal or a leaf node.

## Details 

### Traversal Nodes:
- **Priority Selector:** This node works through the children nodes it contains and will return to its parent if it finds a SUCCESSFUL or RUNNING node.
- **Sequence Selector:** This node works by processing each child node in order, if the currently being process node returns FAILURE or RUNNING it will stop and return to its parent.
- **Concurrent Selector:** This node work by processing all node children, it keeps track of the failed children and if it reaches a "Failure Gate" it will fail. The Failure Gate is configurable based on the Selector definition.

This is just a high level overview of the Traversal nodes used by my platform, and as stated before these selector nodes either call other selector nodes or leaf nodes. Next we will go over the leaf nodes, these nodes a ending nodes and will not contain children.

### Leaf Nodes:
- **Actions:** We can think of these as ways for the BT to act on the Agent.
- **Conditions:** This is a node that will/should only check the state of an Agent or the World to return a SUCCESS/FAILURE response/status.

### Node States

- **Ready:** This node is prepared to process actions or traversal. 
- **Visiting:** This node has been entered and is either processing leaf nodes or process results.
- **Failed:** The node ran a set of pre-determined conditions it deemed as Failed, not to be confused with an Error Status.
- **Running:** The node is either ready to process further or it wants to be queued for future a future check.
- **Success:** The node has finished with a set of pre-determined conditions it deemed as a Success. 
- **Error:** The node has experienced a non-deterministic condition it could not classify.

## Finer Details

### Actions

Actions are a way run mutations on the Actor or the world around them. They might run for one simulation tick, one frame, or might need to be ticked for multiple frames to finish their work. 


TODO: Explain how the Kernel Works
The Kernel is the brains of the of the Actor, it takes in the current/prior state of the Actor and the shape of the behavior tree then running it through the interpreters.

TODO: Explain how the state is setup from prior runs of an Actor

TODO: Explain the Scripting System used

TODO: Explain the platform context that this system is used in

### Example of BT data structure:
~~~ json
{
    "comments": "This will cause the Agent to wander around to random position on the map.",
    "name": "Wander",
    "description": "",
    "root": {
        "type": "SEQUENCE_SELECTOR",
        "nodeList": [
            {
                "comments": "Wait an amount of time before moving on.",
                "type": "ACTION",
                "fire": "Behavior_Action_WaitSomeTime.csx"
            },
            {
                "comments": "This will return a action to populate the Agents move to path.",
                "comments2": "If the path is filled it will",
                "type": "ACTION",
                "fire": "Behavior_Action_FindNewMoveLocation.csx"
            },
            {
                "comments": "This will will return a differed state of Actor movement.",
                "type": "ACTION",
                "fire": "Behavior_Action_MoveToLocation.csx"
            }
        ]
    }
}
~~~


## Terms definitions:

**JSON** - Javascript Object Notation, it is a structured data format.

**BT** - Behavior Tree, the graph of Traversal and Leaf nodes that control an Game Agents state.

## References:

Standard Overview:

***--*** https://www.gamasutra.com/blogs/ChrisSimpson/20140717/221339/Behavior_trees_for_AI_How_they_work.php
Some Really good Blog post on how to implement a Data driven BT:
***--*** https://web.archive.org/web/20131209105717/http://www.altdevblogaday.com/2011/02/24/introduction-to-behavior-trees/
***--*** https://web.archive.org/web/20140217141804/http://altdevblogaday.com:80/2011/04/24/data-oriented-streams-spring-behavior-trees/
***--*** https://web.archive.org/web/20140107101451/http://altdevblogaday.com/2011/07/09/data-oriented-behavior-tree-overview/

Java Implementation: 

***--*** http://magicscrollsofcode.blogspot.com/2010/12/behavior-trees-by-example-ai-in-android.html

Python Implementation:

***--*** https://github.com/andosa/treeinterpreter

Stack Exchange:
***--*** https://gamedev.stackexchange.com/questions/51738/behavior-trees-actions-that-take-longer-than-one-tick?rq=1
***--*** https://gamedev.stackexchange.com/questions/93603/game-ai-behavior-trees-struggles
***--*** https://medium.com/game-dev-daily/advanced-behavior-tree-structures-4b9dc0516f92

Fluent BT:
***--*** https://github.com/ashleydavis/Fluent-Behaviour-Tree
