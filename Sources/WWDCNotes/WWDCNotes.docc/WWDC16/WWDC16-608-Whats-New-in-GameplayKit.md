# What's New in GameplayKit

GameplayKit provides developers a collection of essential tools and techniques used to implement modern gameplay algorithms. Learn what's new in GameplayKit and check out advances in pathfinding, autonomous agents, and game AI, as well as many enhancements supporting GameplayKit in Xcode. Tap into new capabilities for 2D and 3D spatial partitioning, and explore noise-based procedural data generation useful for height maps, natural textures, and more.

@Metadata {
   @TitleHeading("WWDC16")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc16/608", purpose: link, label: "Watch Video (41 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



Updates in:

- Pathfinding
- Agents
- GameAI (formerly known as MinMax AI)

Three new features:

- Spatial partitioning
- Procedural generation
- Xcode integration (with the Xcode editor, not only in code!)

## PathFinding

[`GKMeshGraph`][GKMeshGraph]

Instead of computing line of sight possible paths (like with [`GKObstacleGraph`][GKObstacleGraph]), `GKMeshGraph` uses triangle mesh, where every possible point in the map is represented in one and only one triangle. 

- Less memory intensive, faster to compute, good results (not best tho!)
- Node location flexibility: triangle center, vertices, edges (it can be any of them, even combination of them). The more elements, the more compute intensive

![][PathFinding]
￼
## Agents

- 3D support ([`GKAgent3D`][GKAgent3D] and associated classes) 
- New Behavior composition (similar to weighted goals)

## Spatial Partitioning

Does a spacial caching of the map to answer quicker to inquiries such as:

- How many enemies are near the player? 
- Where are all items in the my world? 
- Which projectiles will hit player this frame?

The way it works is by creating a tree out of the map, the more elements, the more the nodes.

Three possible trees:

- R-trees
- quadtree
- octrees

R-tree is a tree data structure that has a number of hierarchical buckets:

- whenever you add an object to an R-tree, it gets fitted into one of those buckets.
- when these buckets grow too large, they need to split: this is a user-configurable parameter of just how large these buckets can grow
- we have a number of strategies at our disposal to decide how these buckets should split
- all R-trees do is putting things in buckets

![][rtree]

Quadtree and octrees solve the same problem, quad tree on 2D, octrees in 3D:

- these are tree-like data structures that have a number of levels and hierarchies 
- space is subdivided evenly at each level
- have a max cell size associated, this controls just how deep these trees can grow and just how small those cells can get.
- when you add an object to a quadtree or octree, it gets placed into the smallest cell that it fits in entirely

![][rtree2]

When we create a tree, we need to define the minimum size cell: e.g. no cell in this quadtree is going to be smaller than that.  
We’re setting an hard limit on how small (and how many levels deep) our tree can go.

## Game AI

Last year GameplayKit introduced the MinMax Strategist, this year we’re adding Monte Carlo strategist.

This new strategist doesn’t guarantee the best outcome (unlike minMax), but it’s much faster.

This year also we can create our own strategist via [`GKStrategist`][GKStrategist].

New [`GKDecisionTree`][GKDecisionTree]: these are a trees that can be handmade or make the AI learn, and are useful when we have many states and the AI needs to make decisions fast.

## Procedural generation

This is to create new worlds/textures/.. and dynamic things like that, good for games that are not static and need to continuously create new things/levels.

What procedural generation does is help us with some coherent randomness.

Plenty of different noises available, noises can be combined together.

## Xcode integration

Previously everything was done via code, now we can use editors like we do for CoreData and SpriteKit:

- Entity and Components Editor 
- Navigation Graph Editor 
- Scene Outline View
- State Machine Quick Look (directly from the debugger!)

We can use the new callout `@GKInspectable` in our components properties to make them show up in the editor

[PathFinding]: WWDC16-608-PathFinding
[rtree2]: WWDC16-608-rtree-2
[rtree]: WWDC16-608-rtree

[GKAgent3D]: https://developer.apple.com/documentation/gameplaykit/GKAgent3D
[GKDecisionTree]: https://developer.apple.com/documentation/gameplaykit/GKDecisionTree
[GKStrategist]: https://developer.apple.com/documentation/gameplaykit/GKStrategist
[GKMeshGraph]: https://developer.apple.com/documentation/gameplaykit/GKMeshGraph
[GKObstacleGraph]: https://developer.apple.com/documentation/gameplaykit/GKObstacleGraph