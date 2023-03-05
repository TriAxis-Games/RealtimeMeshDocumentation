---
title: Mesh Basics
sections:
  - How Meshes are Represented
  - Vertex Buffer Layout
  - Index Buffer Layout
  - Winding Order
  - Tangents
  - Mesh Hierarchy
---

### How Meshes are Represented

Meshes are represented through a vertex buffer and index buffer. These work in tandem to represent a mesh. The vertex buffer contains a list of all the unique vertices in the mesh, including their position, normal, tangent, color and texture coordinates. The index buffer, in the case of the RMC is in the form of a triangle list. This is laid out as a contiguous list of indices into the vertex buffer 3 at a time representing the 3 points of a triangle.

###### Mesh Structure Example


<div class="screenshot-holder">
  <a href="assets/doc_resources/mesh_basics/TriangleList.svg" data-title="MeshBasics - Structure" data-toggle="lightbox"><img class="img-responsive" src="assets/doc_resources/mesh_basics/TriangleList.svg" alt="Representation of a triangle and it's arrays" /></a>
  <a class="mask" href="assets/doc_resources/mesh_basics/TriangleList.svg" data-title="MeshBasics - Structure" data-toggle="lightbox"><i class="icon fa fa-search-plus"></i></a>
</div>






### Vertex Buffer Layout

The vertex buffer is a list of vertices with all its corresponding data for all the unique vertices in the mesh. There are several options, which can affect the memory consumption, rendering performance, and visual quality of the mesh.

The elements of the vertex buffer are:
1. Position: This contains the position in local space of the vertex of this mesh.
        This is a FVector as it contains a (X, Y, Z) position.
2. Normal/Tangent: This contains the normal and tangent at this vertex and is used for lighting information and materials.
        Normals can be in either normal or high precision format. High precision correllates to the high precision tangents of the engine, so this option is really only useful when the engine is configured to use high precision tangents. The size of the tangents in memory is either 8 bytes for normal position, or 16 bytes for the high precision.
        Normals point up from the triangle towards the camera when the triangle is visible, and tangents point towards the increasing U coordinate of the UV. Cotangents can be calculated from that and point towards the increasing V coordinate of the UV.
        These also determine if you have hard shading (where you can see individual triangles) or soft shading (usually used). For hard shading, each vertex of the tris needs to have the normal of the tris, so that means your vertex buffer is around the same length as the index buffer.
3. Color: This contains a single color value for this vertex, it is up to the material if and how this is used. It can be for vertex painting, or just passing through extra data to the material for other effects.
        This is always represented as FColor, which contains a 4 bytes B, G, R, A, value.
4. Texture Coordinates: Within UE, the mesh can have 1 to 8 channels of texture coordinates, most meshes will only use 1, but you can have all the way up to 8. Beware : most of the time UV 2 is used for lightmaps, so you'll need to configure that correctly.
        There's 2 options here, one is a channel count and the other is a normal/high precision UV's. The datatype switches between FVector2DHalf and FVector2D for normal vs high precision. This means it uses either 4 or 8 bytes of memory per channel, so the texture coordinates can range in memory from 4 bytes for a single channel normal precision up to 64 bytes for all 8 channels in high precision.

Depending on the configuration above, the total vertex size can range from 28 bytes for a normal precision tangents, normal precision texcoord single channel, to as much as 96 bytes for all high precision components with 8 texture coordinate channels.


### Index Buffer Layout

The index buffer is meant to map the vertices into triangles. Within the RMC this takes the form of a triangle list, so indices will be in groups of 3 one after the other to define the 3 points of each triangle, each group of 3 is separate from the others and has no dependence on the others. With that said though order of the vertices within the triangle does matter for culling (See Winding Order below), and order of triangles within the buffer can have performance effects through things like cache coherency, locality, transform cache, and overdraw.

The index buffer can be either a 16 bit integer, or 32 bit integer per element, so it will consume 2 to 4 bytes per element. 16 bit integer is the default as it can handle meshes of up to 2^16 or 65,536 vertices.


### Winding Order

Winding order refers to the order the vertices are reference for the triangle, this is important as one of the most common ways of improving performance is through backface culling. This works by looking at the direction the triangle wraps to detect if it's a front or backface, and if it is found to be a backface then the triangle will get culled in hardware. By default Unreal Engine uses **ClockWise Culling**, so you should make the order of the vertices when referenced in the index list wrap **Counter-ClockWise** when viewed from the visible side. 

It is possible to disable backface culling in Unreal by using two sided materials. If your mesh appears to render inside out, then you need to reverse the winding order.


### Mesh Hierarchy/LOD

Each RealtimeMeshComponent can have 1-8 Levels of Detail or LODs, each of which can have any number of section groups, each of which can have any number of sections. Each LOD is separate from the others, and so can have different numbers of sections and different materials bound to those sections.

Each LOD has a ScreenSize associated to it. This is the percent of the screen the bounding volume has to cover before this LOD is rendered.

### Material Slots

Unlike the ProceduralMeshComponent and old RMC, materials are handled similarly to how StaticMesh handles them. URealtimeMesh holds a number of material slots, setup by SetupMaterialSlot, each has a index, name, and material. You can find these slots by index or name. Each mesh section can be assigned to any slot. 

The RealtimeMeshComponent, like the StaticMeshComponent, has override materials (which was how RMC and PMC previously handled materials). These materials override the slots by index, and allow different components to bind different materials even when they share the same underlying mesh data.