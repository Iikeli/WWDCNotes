# Introducing GameplayKit

GameplayKit provides a collection of essential tools and techniques used to implement gameplay logic. Get introduced to the GameplayKit framework and see how to put its capabilities to work in your own titles. Learn about managing state machines, controlling game entities, and implementing rule systems. Dive into its built-in tools for randomization, pathfinding, and advanced simulation.

@Metadata {
   @TitleHeading("WWDC15")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc15/608", purpose: link, label: "Watch Video (52 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## What is GameplayKit?

GameplayKit is a high-level gameplay framework, it consists of a collection of common architectural patterns, data structures, and algorithms that enables our developers to make really great and compelling gameplay in their games.

Seven major features:

- Entities & Components - great way to structure your game objects and game logic
- State Machines - describe the statefulness in our games and the various state changes of our game objects
- Agents - autonomously moving entities that are controlled by realistic behaviors and goals
- Pathfinding - deals with navigation graph generation and how we move our entities between the passable areas in our game world
- MinMax AI - great way to give life to our computer-controlled opponents
- Random Sources 
- Rule Systems - great way to model discreet and fuzzy logic

## Entities & Components

Image we’re building a tower defense game:  
we have multiple entities in our game, instead of them having functionality in an inheritance sense, e.g. being a mover, being a shooter, or being targetable, they instead have components, which encapsulate singular elements of our game logic. 

For example, we have a `MoveComponent` that deals with moving, a `ShootComponent` that deals with shooting, and a `TargetComponent`, bringing the meaning of being targetable.

Using components scales very well with complexity, you can have different people working in different components with no issue.

Every time we create a new entity, we can reuse components that we have already defined or create new ones.

If we need a finer control over the order or how your components update, we can use [`GKComponentSystem`][GKComponentSystem]: this is a collection of components from different entities, but they're all the same class type.

For example, if we want the `AI` component to always update after the `move` component, we can set so via `GKComponentSystem`.

> The components within a `GKComponentSystem` no longer update when their entities call update, it’s up to us to call update on the system instead.

## State Machines 

Used to define the possible states of an entity, and also what transitions are possible from each state.

If we implement this manually, we might define a state as an `enum` with multiple cases, however managing each transition generates lots of boilerplate (big `switch` statements, deep `if-else` trees), and the more you use the pattern within the app, the more boilerplate we generate.

GameplayKit state machines lets us:

- remove that boilerplate
- improve maintainability
- give us the benefit of being able to reuse our states and state machines throughout our game

[`GKStateMachine`][GKStateMachine] is your general purpose finite state machine. It's always in one, and only one state at any given time. It possesses (and know) all the possible states.

We call `enterState:` on our state machines to cause a state transitions (this is also how we define the initial state of the machine): under the hood, it checks if that transition is valid, and if so, makes that change.

>  This is possible to control by overriding `isValidNextState:`

The state machine calls a number of call backs on the state objects.

## Agents, Goals, and Behaviors

These are autonomously moving entities, they're controlled by realistic behaviors and goals. They have a number of physical constraints, things like masks, acceleration, and inertia. 

The idea is to make our entities more naturally: e.g.  if our game entities move in straight lines and take sudden turns and bump into environment obstacles, it doesn't look very real/natural. Movement in the real world has things like inertia, and mass, and acceleration. These agents help us reaching reality in our games.

- [`GKAgent`][GKAgent] (which is also a [`GKComponent`][GKComponent])
- [`GKBehavior`][GKBehavior] we can add/remove goals
- [`GKGoal`][GKGoal] the focus of an action

An example of usage of this feature is making a racing game and defining the other drivers as agents, where the behavior is the sum of three goals: following the path (1) while also avoid obstacles (2) and do it as fast as possible (3). 

We can assign different weights to each goal, so that the final behavior changes based on the weight as well.

The session has a demo for Agents/Goals/behaviors where:

- first a ship has to chase the mouse pointer
- then the ship has to run away from it
- then the ship simply wanders around
- then another one with obstacles

The demo always uses the same logic, but with different behavior. All movements are very natural from the ship just by using these components.

More cool demos: multiple ship chasing the pointer, with also the goal to avoid collisions.

## Pathfinding

Find the optimal path between two nodes within a navigation graph (bidirectional, omnidirectional, or mixed).

- [`GKGraph`][GKGraph] - Abstract graph base class, it's a container of graph nodes. 
- [`GKGridGraph`][GKGridGraph] - 2D grid `GKGraph` specialization: 
  - automatically generates all the nodes to represent a grid of some given start position, width, and height
  - automatically makes the cardinal connections between the grid nodes and optionally the diagonal ones as well.
  - it's possible to add/remove grid spaces 

- [`GKObstacleGraph`][GKObstacleGraph] - Pathfinding around obstacles `GKGraph` specialization:
  - Under the hood, this is still a 2D graph. After defining our obstacles (and their buffer area), the framework is going to make the appropriate connections between all of our grid nodes, and it's going to correctly _not_ make the ones that would violate the spatiality of our obstacles

![][obstacles]

If we use SpringKit, we can easily generate obstacles from [`SKNode`][SKNode] bounds, physics bodies, or textures

```objc
/* Makes obstacles from sprite textures */
(NSArray*)obstaclesFromSpriteTextures:(NSArray*)sprites accuracy:(float)accuracy;

/* Makes obstacles from node bounds */
(NSArray*)obstaclesFromNodeBounds:(NSArray*)nodes;

/* Makes obstacles from node physics bodies */
(NSArray*)obstaclesFromNodePhysicsBodies:(NSArray*)nodes;
```

## MinMax AI 

Many games need equal AI opponents (mimicking a human);

- Can play the entire game
- Play by the same rules as human players

Think games such as Chess, Checkers, Tic-Tac-Toe, etc.

What MinMax AI does:

- Looks at player moves
- Builds decision tree 
- Maximizes potential gain 
- Minimizes potential loss 

Features

- AI-controlled opponents
- Suggest move for human players
- Best suited for turn-based games
  - Any game with discrete moves

- Variable difficulty
  - Adjust look ahead
  - Select sub-optimal moves

The great thing about MinMax is that it doesn't need to know any of the details of your game. You don't need to teach it your rules and it doesn't need to know how it's implemented. This is all abstracted away. All you have to do is provide a list of players in the game, the possible moves they can make, and a score for each player that indicates the relative strength of their current position.

When you request a move from the AI, it takes all this data into account and it builds a decision tree, and returns the optimal move for you to use.

There are three key protocols that you're going to need to implement to work with the MinMax AI.

- [`GKGameModel`][GKGameModel] - abstract of the current game state  
  This needs to know the current game state (if this was a chess game, it’d need to know the whole Chess board state, the current player turn, and which are the possible moves.

- [`GKGameModelUpdate`][GKGameModelUpdate] - abstraction of a move within your game  
  It should have all of the data you need to apply a move to your game model

- [`GKGameModelPlayer`][GKGameModelPlayer] - abstraction of a player of the game
  It's used by the AI to differentiate moves from one another

## Random Sources 

Different possible randomizations/distributions

## Rule Systems

A game consists of three elements: 

- Nouns (Properties) - Position, speed, health, equipment, etc. 
- Verbs (Actions that the player does) - Run, jump, use item, accelerate, etc. 
- Rules - How your nouns and verbs interact 

[obstacles]: WWDC15-608-obstacles

[GKObstacleGraph]: https://developer.apple.com/documentation/gameplaykit/GKObstacleGraph
[GKGridGraph]: https://developer.apple.com/documentation/gameplaykit/gkgridgraph
[GKGameModelUpdate]: https://developer.apple.com/documentation/gameplaykit/GKGameModelUpdate
[GKGameModelPlayer]: https://developer.apple.com/documentation/gameplaykit/GKGameModelPlayer
[GKGameModel]: https://developer.apple.com/documentation/gameplaykit/GKGameModel
[SKNode]: https://developer.apple.com/documentation/spritekit/sknode
[GKGraph]: https://developer.apple.com/documentation/gameplaykit/GKGraph
[GKGoal]: https://developer.apple.com/documentation/gameplaykit/GKGoal
[GKBehavior]: https://developer.apple.com/documentation/gameplaykit/GKBehavior
[GKAgent]: https://developer.apple.com/documentation/gameplaykit/GKAgent
[GKComponent]: https://developer.apple.com/documentation/gameplaykit/GKComponent
[gkstatemachine]: https://developer.apple.com/documentation/gameplaykit/gkstatemachine
[GKComponentSystem]: https://developer.apple.com/documentation/gameplaykit/gkcomponentsystem
