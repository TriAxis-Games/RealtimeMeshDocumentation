---
title: Using the RMC like the PMC (URealtimeMeshSimple)
sections:
  - What is the RealtimeMeshSimple ?
  - Adding it to your code
  - Giving it mesh data
  - Giving it collision
  - Extra features
---


### What is the RealtimeMeshSimple?
The URealtimeMeshSimple, is an implementation of the URealtimeMesh that provides a very simple, ProceduralMeshComponent like, way of using the RMC. It provides full support for LODs, SectionGroups, and Sections, like any other imlementation, but it behaves like the ProceduralMeshComponent. It allows you to feed it data, and it will manage it from there requiring no more input or extra support to handle.

### Adding it to your code
Add `#include "RealtimeMeshSimple.h` to the header or .cpp of the class you want to use it in. Then, you can create it like any static mesh component or the likes. 

The fastest way to then get it running is like this...
```cpp
	URealtimeMeshSimple* RealtimeMesh = GetRealtimeMeshComponent()->InitializeRealtimeMesh<URealtimeMeshSimple>();
```
This will Initialize the URealtimeMeshSimple and set it to the RMC all in one shot.

You can also do `NewObject<URealtimeMeshSimple>()` like any other UObject and then pass it to one or more RealtimeMeshComponents

### Giving it mesh data in C++
The first thing you'll need is to fill a `FRealtimeMeshSimpleMeshData` with your mesh data. This is simply a collection of the various arrays for Position, Normal, Tangent, TexCoords, Colors, and Triangles.


The simplest way to setup a drawable section is to use CreateMeshSection or one of its variants.
```cpp
	FRealtimeMeshSectionKey StaticSectionKey = RealtimeMesh->CreateMeshSection(0,
		FRealtimeMeshSectionConfig(ERealtimeMeshSectionDrawType::Static, 0), MeshData, true);
```
In this example, we're creating a static draw path section, in LOD 0, with the supplied MeshData and we want to create mesh collision for the section. This will return a FRealtimeMeshSectionKey which can be used later to Update/Clear/Remove or any other operation on the section.

Then to update the mesh data you can simply call UpdateSectionMesh and give it the key returned from CreateMeshSection, as well as the new MeshData
```cpp
	RealtimeMesh->UpdateSectionMesh(StaticSectionKey, MeshData);
```

### Advanced setup
The above steps are mostly the same, with the difference being you will create a SectionGroup first and create multiple Sections within it. This allows setting one large set of buffers for all the mesh data, and having different sections share it. This can be used for everything from simply reducing data/state transitions, to allowing sections to share vertex data and rendering different subsets, or overlapping regions etc.

First you'll setup the section group, and pass it the mesh data.
```cpp
	FRealtimeMeshSectionGroupKey GroupKey = RealtimeMesh->CreateSectionGroupWithMesh(0, MeshData);
```
This will create a section group with the supplied mesh data and return the FRealtimeMeshSectionGroupKey you can use to operate on the group or add sections to, or remove sections from, the group.

```cpp
	FRealtimeMeshSectionKey SectionInGroup = RealtimeMesh->CreateSectionInGroup(GroupKey,
		FRealtimeMeshSectionConfig(ERealtimeMeshSectionDrawType::Static, 0),
		FRealtimeMeshStreamRange(0, 24, 0, 36), true);
```
Here we pass it the group key, and the basic section config again. The difference here is now we pass it a stream range, that specifies the inclusive lower bound, and the exclusive upper bound of the vertex and index buffer set in the section group that this section will be rendering from.

You can update the whole Section Groups mesh data with this call
```cpp
	RealtimeMesh->UpdateSectionGroupMesh(SectionInGroup, MeshData);
```

