---
title: Collision
sections:
  - Simple Collision
  - Complex Collision
  - Settings
---

Collision in the RMC has a few different parts, including basic settings and two collision types each of which has its own benefits and limitations.

### Simple Collision

Simple collision is made up of Boxes, Spheres, Cylinders, and Convex Elements. These are the basis of simple collision, which is required for movable objects that can interact with the environment and perform overlap tests. These are all convex shapes, as concave collision detection is far more complex. You can have none, one, or multiple of each of these 4 shapes, but you must have at least one to have any form of simple collision/physics simulation.
Convex means that any line between two points belonging in the volume belongs to the volume.

Convex Elements are a convex mesh object. These can be generated directly by you, or possibly through a process known as convex decomposition where you take a source mesh and generate one or more convex shapes to represent it. This is how UE4 handles arbitrary shapes for collision.
Convex elements are slower to compute than other primitives, and are limited to 256 vertices.

Note : Collisions don't have to be perfect, since the primitives are invisible. Sacrificing accuracy for performance is alright : Avoid using convexes.

### Complex collision

Complex collision is made from a triangular mesh. This can be either from your renderable mesh data, or a custom simplified mesh for collision. Usually the latter is better for collision performance, but takes extra effort to generate. No two complex collision shapes can perform collision tests, so complex collision only objects are not allowed to simulate physics.
Having a line trace set to complex will return the complex collision mesh's triangle index on hit.

### Settings

The collision settings object is used to set the simple collision shapes, as well as some basic collision cooking settings.

Collision Cooking is required for convex elements and complex mesh. This process builds internal structures for performing high performance collision, but it takes a non-trivial amount of time to perform this collision on complex meshes. With this the RMC supports Async Cooking which can be turned on or off through the flag bUseAsyncCooking.

Complex as simple collision is where you have no simple collision for a shape and let the complex collision be used for things such as line tracing which would normally use the simple collision.
This setting is controlled by bUseComplexAsSimple on the collision settings object.

CookingMode can be set to either CollisionPerformance or CookingPerformance. The first prioritizes efficient collision detection at the cost of a little more effort in cooking to build optimal collision structures. The second prioritizes quickly cooking meshes over having optimal collision structures. 
