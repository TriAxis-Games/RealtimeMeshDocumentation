---
title: Using the RMC like the PMC (RuntimeMeshComponentStatic)
sections:
  - What is the RuntimeMeshComponentStatic ?
  - Adding it to your code
  - Giving it mesh data
  - Giving it collision
  - Extra features
---


### What is the RuntimeMeshComponentStatic ?
The RuntimeMeshComponentStatic, or RMCS or RMCStatic, is a component made to replace the ProceduralMeshComponent (PMC) by providing the same kind of functions as the PMC while giving the advantages of the RMC. For instance, it allows LODs, mesh sharing, lower level access to collision and different mesh cooking modes.
It is named static because it directly exposes the functions found inside the StaticProvider, and creates the provider automatically for you.
Note : The RMCS can only share it's mesh data with other RMCS (or RMC, as long as they don't mind having the same provider structure); when sharing, you only need to give the mesh data to one component, the other will be updated automatically; giving it multiple times will only waste resources.
Note : RMCs is RuntimeMeshComponents, plural.

### Adding it to your code
Add `#include "Components/RuntimeMeshComponentStatic.h` to the header or .cpp of the class you want to use it in. Then, you can create it like any static mesh component or the likes.

### Giving it mesh data
If you're using the RMCS from Blueprint, you have two choices : Eiher give the mesh from it's components using `Create Section From Components`, or give it using `Create Section`. The later requires you to create a `Runtime Mesh Renderable Mesh Data`, for that use `Create Renderable Mesh Data`, and then you can access all of it's streams using `Get <name of the stream> Stream`. See `Source/RuntimeMeshComponent/Public/RuntimeMeshBlueprintFunctions.h` for more details.
If you want to change he mesh data instead, you can call the functions with the same name but Create is replaced with Update.
You can remove a section by calling `Clear Section`.

If you use the RMCS from C++, the functions are the same, but you have more variants :

    void CreateSection_Blueprint(int32 LODIndex, int32 SectionId, const FRuntimeMeshSectionProperties& SectionProperties, const FRuntimeMeshRenderableMeshData& SectionData);
    void CreateSectionFromComponents(int32 LODIndex, int32 SectionIndex, int32 MaterialSlot, const TArray<FVector>& Vertices, const TArray<int32>& Triangles, const TArray<FVector>& Normals,
		const TArray<FVector2D>& UV0, const TArray<FVector2D>& UV1, const TArray<FVector2D>& UV2, const TArray<FVector2D>& UV3, const TArray<FLinearColor>& VertexColors,
		const TArray<FRuntimeMeshTangent>& Tangents, ERuntimeMeshUpdateFrequency UpdateFrequency = ERuntimeMeshUpdateFrequency::Infrequent, bool bCreateCollision = true);
	void CreateSectionFromComponents(int32 LODIndex, int32 SectionIndex, int32 MaterialSlot, const TArray<FVector>& Vertices, const TArray<int32>& Triangles, const TArray<FVector>& Normals,
		const TArray<FVector2D>& UV0, const TArray<FVector2D>& UV1, const TArray<FVector2D>& UV2, const TArray<FVector2D>& UV3, const TArray<FColor>& VertexColors,
		const TArray<FRuntimeMeshTangent>& Tangents, ERuntimeMeshUpdateFrequency UpdateFrequency = ERuntimeMeshUpdateFrequency::Infrequent, bool bCreateCollision = true);
	void CreateSectionFromComponents(int32 LODIndex, int32 SectionIndex, int32 MaterialSlot, const TArray<FVector>& Vertices, const TArray<int32>& Triangles, const TArray<FVector>& Normals,
		const TArray<FVector2D>& UV0, const TArray<FLinearColor>& VertexColors, const TArray<FRuntimeMeshTangent>& Tangents,
		ERuntimeMeshUpdateFrequency UpdateFrequency = ERuntimeMeshUpdateFrequency::Infrequent, bool bCreateCollision = true);
	void CreateSectionFromComponents(int32 LODIndex, int32 SectionIndex, int32 MaterialSlot, const TArray<FVector>& Vertices, const TArray<int32>& Triangles, const TArray<FVector>& Normals,
		const TArray<FVector2D>& UV0, const TArray<FColor>& VertexColors, const TArray<FRuntimeMeshTangent>& Tangents,
		ERuntimeMeshUpdateFrequency UpdateFrequency = ERuntimeMeshUpdateFrequency::Infrequent, bool bCreateCollision = true);
Same is true for updating.
Note : When updating, it doesn't need setup data, so the parameters are slightly different.

Note : FRuntimeMeshRenderableMeshData is the way the RMC handles meshes internally, so it is very inconvenient to use without functions to add vertices or triangles. Avoid using it directly from BP as each function call has a fixed time cost. Best prqctice would be to make your mesh in C++ using a library of some sort.

### Giving it collision
For simple collision, all you need is to call `void SetCollisionSettings(const FRuntimeMeshCollisionSettings& NewCollisionSettings);` (Search for `Set Collision Settings in BP`).
If you want complex collision, you can get the complex collision mesh from a renderable LOD, by using `void SetRenderableLODForCollision(int32 LODIndex);` and you can add or remove sections that should be included in the complex collision using `void SetRenderableSectionAffectsCollision(int32 SectionId, bool bCollisionEnabled);`
You can also choose to handcraft the complex collision mesh and give it to the RMCS using `void SetCollisionMesh(const FRuntimeMeshCollisionData& NewCollisionMesh);`, the way you build that collision mesh is fairly similar to building a visible mesh, except it doesn't some of the data.

### Extra features
The bounds for the mesh will be computed automatically from the mesh.
You can choose to have normals and tangents computed for you using :

    UFUNCTION(BlueprintCallable, Category = "RuntimeMeshStatic|Normals")
	void EnableNormalTangentGeneration();

	UFUNCTION(BlueprintCallable, Category = "RuntimeMeshStatic|Normals")
	void DisableNormalTangentGeneration();

	UFUNCTION(BlueprintCallable, Category = "RuntimeMeshStatic|Normals")
	bool HasNormalTangentGenerationEnabled() const;

If you need tesselation, you can generate the needed triangles using :

	UFUNCTION(BlueprintCallable, Category = "RuntimeMeshStatic|Tessellation")
	void EnabledTessellationTrianglesGeneration();

	UFUNCTION(BlueprintCallable, Category = "RuntimeMeshStatic|Tessellation")
	void DisableTessellationTrianglesGeneration();

	UFUNCTION(BlueprintCallable, Category = "RuntimeMeshStatic|Tessellation")
	bool HasTessellationTriangleGenerationEnabled() const;