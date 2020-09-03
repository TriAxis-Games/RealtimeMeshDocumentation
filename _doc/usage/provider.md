---
title: Building your own provider
sections:
  - Creating the class
  - Provider setup
  - Updating the parameters
  - Making a visible mesh
  - Making a collidable provider
  - Using LODs
  - Using it in the RuntimeMesh(Component)
---

This tutorial will show how to make a multithreadable provider, so that once it is done, the only thing you'll need to care about are the arguments given to it, and no longer the mesh.

## Creating the class
You can start by copying one of the builtin providers, such as the cube or sphere, as these will come with all of the most important functions, whereas generating the class will give you no functions, leaving you to paste them.

## Provider setup
When the provider is first initialized, `void Initialize() { }` will be called. This is where you'll need to declare all needed materials, LODs and sections, declared in this order preferably. 
For the materials, you don't actually set them up, instead you setup a slot to receive a material and then you attach section to use it, using `void SetupMaterialSlot(int32 MaterialSlot, FName SlotName, UMaterialInterface* InMaterial)`.
For the LODs, you might declare at which point they no longer render, expressed in screen percentage, after which the LOD with a higher index will render instead, or if it doesn't exist, it won't render at all; using `void ConfigureLODs(const TArray<FRuntimeMeshLODProperties>& InLODs)`.
For the sections, you can specify a lot of things, most important being to use either 16bit or 32bit indices and the update frequency. This is needed when you have more than 2^16 (65536) vertices. Use `void CreateSection(int32 LODIndex, int32 SectionId, const FRuntimeMeshSectionProperties& SectionProperties)`.
Note : You can set if 32bit indices are needed later, when starting to generate the mesh, by recreating the triangle stream.
Note : Update frequency will change the time taken to render and the time taken to make the mesh, Frequent taking more time to make the mesh but less time to render, and Infrequent taking less time to make the mesh but more time to render. No testing has been done on the real improvements given by these different render path, feel free to try it and tell us the results ! Ping me on the discord, @Moddingear#8281

## Updating the parameters
You'll need to create a getter and a setter for each parameter, if you wish to be able to modify them individually. You'll also need to have a class-wide scope lock, in order to use multithreading. Here's a example, using the box provider :

RuntimeMeshProviderBox.h

    class RUNTIMEMESHCOMPONENT_API URuntimeMeshProviderBox : public URuntimeMeshProvider
    {
    	GENERATED_BODY()
    private:
    	mutable FCriticalSection PropertySyncRoot;
    	FVector BoxRadius;
    public:
		FVector GetBoxRadius() const;
		void SetBoxRadius(const FVector& InRadius);
    	/*there's other stuff after but that's not the point*/
    }

RuntimeMeshProviderBox.cpp

    FVector URuntimeMeshProviderBox::GetBoxRadius() const
    {
    	FScopeLock Lock(&PropertySyncRoot);
    	return BoxRadius;
    }
    
    void URuntimeMeshProviderBox::SetBoxRadius(const FVector& InRadius)
    {
    	FScopeLock Lock(&PropertySyncRoot);
    	BoxRadius = InRadius;
    
    	MarkAllLODsDirty();
    	MarkCollisionDirty();
    }
   
Note : You can call any setup function you like, and calling the setter before initialization will not cause any problems, as the provider is not connected to the RMC, so the setup calls will just be dropped.
Note : `void MarkCollisionDirty();` `void MarkAllLODsDirty();` `void MarkLODDirty(int32 LODIndex)` are used to force an update on the RMC, in order to tell it that you want the mesh data to be fetched again.

## Making a visible mesh
I'll assume you've read the quickstart guide, if not, click [here]() **TODO ADD LINK**.
A mesh needs two things : Bounds, and renderable data.
Bounds should fit your mesh tightly. If they are too big, you'll waste resources drawing something invisible. If they are too small, the mesh will pop into existence. Try to find some simple way to compute them, then the RMC will call `FBoxSphereBounds GetBounds()` to get them.
For the renderable data, the RMC will call `bool GetSectionMeshForLOD(int32 LODIndex, int32 SectionId, FRuntimeMeshRenderableMeshData& MeshData)` or `bool GetAllSectionsMeshForLOD(int32 LODIndex, TMap<int32, FRuntimeMeshSectionData>& MeshDatas)`, depending on the LOD configuration. Unless there's an error, you should return true. This call may be made out of the game thread, so you should always use the scope lock to get the needed parameters. Tbh you should always use the scope lock when getting the parameters, just in case.
Note : Some functions may help you to make the mesh, for example the RuntimeMeshModifierNormals can compute the normals and tangents of the mesh.

## Making a collidable provider
In Unreal, there are two types of collidables : simples and complex. Simples can be added to the provider when requested by the RMC, when it calls `FRuntimeMeshCollisionSettings GetCollisionSettings()`.
Complexes need to be enabled by returning true to `bool HasCollisionMesh()` when called by the RMC, and then giving a mesh to `bool GetCollisionMesh(FRuntimeMeshCollisionData& CollisionData)`. The mesh itself is slightly different in structure from the renderable one.
Note : There's an array called CollisionSources in the CollisionData, that's for debugging purposes, to tell what the collision mesh was generated by.
Note : RuntimeMeshProviderCollision can turn a renderable LOD/section into a collidable mesh, it's a passthrough provider.

## Using LODs

Same as with making a visible mesh, except you use the LODIndex given to you when the function is called by the RMC.
That's the beauty of providers.

## Using it in the RuntimeMesh(Component)

Create the provider (it's a UObject), give it the parameters you want, then call RMC->Initialize(Provider);
You can store the pointer to the provider to update the parameters later if you need to
