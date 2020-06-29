---
title: Builtin Providers
sections:
  - ProviderStatic
  - ProviderBox
  - ProviderCollision
  - ProviderMemoryCache
  - ProviderModifiers
  - ProviderPlane
  - ProviderSphere
  - ProviderStaticMesh
---

### ProviderStatic

URuntimeMeshProviderStatic is probably the most commonly used provider. It providers ProceduralMesh/RMCv3 style functionality where you can set the mesh data from outside the component and forget about it. It provides full collision support and has the ability to directly host URuntimeMeshModifiers that get run when you create/update a section.

### ProviderBox

URuntimeMeshProviderBox is a simple provider capable of creating a single box of specified dimensions. This is also the simplest provider to look at when wanting to understand how to make your own.

### ProviderCollision

URuntimeMeshProviderCollision is a pass through provider where you link it to another provider and the collision provider will create the collision mesh data from the renderable sections of the linked provider.

### ProviderMemoryCache

URuntimeMeshProviderMemoryCache is meant to cache mesh data in memory when you make your own provider and want to have the mesh data cached across calls from the URuntimeMesh. Usually this isn't too necessary, unless your provider is especially heavy and you don't want to regenerate all sections of a static batch when one updates.

### ProviderModifiers

URuntimeMeshProviderModifiers is a pass through provider where you link your provider and then can set one or more URuntimeMeshModifiers to alter the mesh data as it's passed through to the URuntimeMesh. This is not necessary with the URuntimeMeshProviderStatic as that provider can hose URuntimeMeshModifiers directly.

### ProviderPlane

URuntimeMeshProviderPlane is a simple provider to make a single plane at one or more levels of detail.

### ProviderSphere

URuntimeMeshProviderSphere is a simple provider to make a single sphere at one or more levels of detail.

### ProviderStaticMesh

URuntimeMeshProviderStaticMesh is a basic provider to render a UStaticMesh through the RMC. This doesn't however give you the ability to modify the mesh data unless you make a custom pass through provider.