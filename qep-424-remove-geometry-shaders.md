# QGIS enhancement: Move away from geometry shaders in QGIS 3D

**Date** 2026-04-13

**Author** Martin Dobias (@wonder-sk)

**Contact** wonder dot sk at gmail dot com

**Version** QGIS 4.4

## Summary

Some of the existing materials in QGIS 3D use geometry shaders in addition to vertex and fragment shaders. In simple terms, geometry shaders are used in QGIS to build triangular mesh from other kind of geometry (e.g. points or lines). The problem with geometry shaders is that the graphics community nowadays considers them as a legacy, and are best to be avoided.

Geometry shaders seem to be a common GPU pipeline bottleneck, because they do not work well with GPU parallelism. They are also harder to understand and to debug. Modern graphics APIs do not always support them: there are no geometry shaders in Metal or WebGPU (not WebGL), and their support in Vulkan is optional. Also Qt's QRhi abstraction does not support geometry shaders, shall we consider switching the Qt3D from OpenGL to RHI (or even replace Qt3D by direct QRhi usage).

Moving away from geometry shaders should therefore future-proof QGIS 3D implementation, and potentially also increase the performance (although that is not the main concern here).

## Proposed Solution

There are actually three materials with geometry shaders:

  1. **Billboards** (src/3d/shaders/billboards.geom). Expands each input point into a camera-facing quad (4 verts). Replacement should be easy, using GPU instancing. We will upload a static unit quad (4 verts, 2 tris) once, then instance per billboard, using per-instance attributes. All the math to the vertex shader, using gl_VertexID (0..3) to pick the corner.

  2. **Lines (including polygon edges, rubber bands)** (src/3d/shaders/lines.geom). Takes lines_adjacency (4 verts: prev / p1 / p2 / next), does near-plane Cohen-Sutherland clipping, screen-space miter join, falls back to butt cap + fill triangles when
  miter limit is exceeded. Emits 4-7 verts per segment. The replacement will be more difficult. The standard modern technique is to use instanced quads per each line segment. Line joins are handled either with the quads, or with another instanced pass to patch the joins (used for round joins - but we do not support them for now). Near-plane clipping may get a bit tricky.

  3. **Mesh layer wireframes** (src/3d/shaders/mesh/mesh.geom).  Used for robust wireframe drawing on top of the mesh. Replacing it should be medium effort. We can calculate barycentric coordinates for the wireframe in the fragment shader.

## Deliverables

The deliverables should be pull requests that remove the geometry shaders and rework materials (and related vertex data preparation).

## Risks

There is a small risk of getting stuck in implementation of lines material given the complexity... while the overall implementation approach is clear, I do not have a prototype yet. On the other hand, there are various other projects using the same technique, in case we needed some inspiration.

## Performance Implications

There may be potentially increased GPU memory use (depending on the exact implementation choices), but overall the rendering performance should be better (at least not worse!)

## Backwards Compatibility

N/A
