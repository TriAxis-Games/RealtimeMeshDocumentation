---
title: Component Structure
sections:
  - URuntimeMeshComponent
  - URuntimeMesh
  - URuntimeMeshProvider
  - URuntimeMeshModifier
---

The Runtime Mesh Component is made up of several distinct parts, each of which provides a portion of the overal system. Together the combination can be very extensible, or very simple. If you're familiar with how UStaticMesh and UStaticMeshComponent works, this will be very similar with a couple extra pieces.


### URuntimeMeshComponent

URuntimeMeshComponent is the main component that allows you to place a URuntimeMesh in the scene and interact with it exactly like UStaticMeshComponent. It's possible to have multiple URuntimeMeshComponents all sharing a single URuntimeMesh, this means that a single copy of the gpu buffers can be shared among several individual components that all can act independently while drawing the same mesh. This is not the same as instancing which draws multiple copies of the mesh at different locations/rotations within the same component, but they all function together.

### URuntimeMesh

URuntimeMesh is the data carrier component. It is responsible for owning the GPU buffers, and the physics object that can then be used by one or more URuntimeMeshComponent's.  When you update the mesh data of a URuntimeMesh all the linked URuntimeMeshComponents will receive the mesh update together and they will all start rendering the new mesh. The URuntimeMesh **DOES NOT** store any mesh data in main memory, it simply copies it out to gpu memory (VRAM) and then destroys its cpu side copy. For collision it will pass the data to PhysX/Chaos and once the cooked collision shapes are created then it also removes the copy of the raw mesh data for collision as it's no longer necessary. It relies on the URuntimeMeshProvider to get any of this data again if it needs it.

### URuntimeMeshProvider

URuntimeMeshProvider is the base class for the provider system. The provider system is how a URuntimeMesh gets the configuration, renderable mesh data, and collision settings/data.  URuntimeMeshProviders' are composable so they can be chained together. For example if you create your own provider that only generates the renderable data, then you can use URuntimeMeshProviderCollision with it to generate the collision data from your rendering data. A URuntimeMesh must have a valid URuntimeMeshProvider to function, but that provider can be either a single provider or a chain of providers. This will in the future also allow the RMC to decide when to generate additional LODs instead of having to create them all upfront as you do now.

### URuntimeMeshProviderPassthrough

URuntimeMeshProviderPassthrough is a base class for pass through providers. These work by linking one or more providers to them and doing internal logic on the returned results from the linked providers before passing the result on to the URuntimeMesh.

### URuntimeMeshModifier

URuntimeMeshModifier is the base class for the modifier systems. Some providers support applying generic modifiers to them that can *modify* the mesh data as it passes through the provider. These are allowed to do things such as clean mesh data, or generate the extra index buffers, but they're not allowed to generate mesh data directly, that is up to the provider. These are simply for common functionality that you'd rather not implement yourself. There is a URuntimeMeshProviderModifiers that can be placed between your provider and the URuntimeMesh to apply these, and the URuntimeMeshProviderStatic has a built in modifier support.

