---
title: Builtin Modifiers
sections:
  - ModifierNormals
  - ModifierAdjacency
---

### ModifierNormals

URuntimeMeshModifierNormals can generate normals and tangents for input mesh data. 
You can also call this modifier directly in your own code by using the static function URuntimeMeshModifierNormals::CalculateNormalsTangents()


### ModifierAdjacency

URuntimeMeshModifierAdjacency can generate the adjacency index buffer need for tessellation. This generates the 12 control point patch list for PN-AEN triangles with dominant corner.
You can also call this modifier directly in your own code by using the static function URuntimeMeshModifierAdjacency::CalculateTessellationIndices()