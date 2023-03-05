---
title: Component Structure
sections:
  - URealtimeMeshComponent
  - URealtimeMesh
  - URealtimeMeshSimple
  - URealtimeDynamicMesh
  - URealtimeMeshComposable
---

The Realtime Mesh Component is made up of several distinct parts, each of which provides a portion of the overall functionality. Together the combination can be extremely extensible, or very simple. If you're familiar with how UStaticMesh and UStaticMeshComponent works, this well be a simliar configuration, with several things built on top.


### URealtimeMeshComponent

URealtimeMeshComponent is the main component that allows you to place a URealtimeMesh in the scene and interact with it exactly like UStaticMeshComponent. It's possible to have multiple URealtimeMeshComponents all sharing a single URealtimeMesh, this means that a single copy of the gpu buffers can be shared among several individual components that all can act independently while drawing the same mesh. This is not the same as instancing which draws multiple copies of the mesh at different locations/rotations within the same component, but they all function together.

### URealtimeMesh

URealtimeMesh is the data carrier component, and is abstract on its own as it relies on a concrete implementation to fully function. It is responsible for owning the GPU buffers, and the physics object that can then be used by one or more URealtimeMeshComponents.  When you update the mesh data of a URealtimeMesh all the linked URealtimeMeshComponents will receive the mesh update together and they will all start rendering the new mesh. The base URealtimeMesh does not store any data in main memory, it simply sends it over to and stores it in VRAM to be used by the rendering pipeline. For collision a similar situation is used where the concrete implementations of URealtimeMesh decides where it get the data and whether to store it.

### URealtimeMeshSimple

URealtimeMeshSimple is the most direct and simple to use implementation of URealtimeMesh. It works exactly like the ProceduralMeshComponent with added features. You setup your LODs, setup your Section Groups, and setup your Sections, and provide them mesh data and the URealtimeMeshSimple takes it from there, storing all the mesh data for subsequent re-use so you can forget about it from there forward.

### URealtimeDynamicMesh

URealtimeDynamicMesh is the next implementation of URealtimeMesh, and it provides a way of rendering UDynamicMesh objects through the RMC. This lets you use the powerful geometry scripting systems found in Unreal Engine 5, while also being able to use the improved rendering performance and additional features of the RMC.

### URealtimeMeshComposable

URealtimeMeshComposable is the third implementation of URealtimeMesh that uses a provider stack to get the mesh data. You can chain providers together to compartmentalize logic, and the URealtimeMeshComposable will handle requesting the mesh data as it's needed. The URealtimeMeshComposable doesn't store the mesh data itself, so for example if you have a caching provider that pages to disk it can pull it in the rare case it needs it, but doesn't keep it in vram.  This is similar to the older RMC's provider stack except that a provider chain can be shared amongst several URealtimeMeshComposables to support having a factory stype setup that can create multiple components.

