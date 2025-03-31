# QGIS Enhancement: Adopt `wasm32-emscripten` as a build target for QGIS

**Date** 2025/03/10

**Author** Michael Schmuki (@boardend )

**Contact** michael at opengis dot ch

**Version** QGIS 3.44

# Summary

[qgis-js](https://github.com/qgis/qgis-js) has demonstrated that it is possible to cross-compile QGIS core and it's dependencies with [Emscripten](https://emscripten.org/) to [WebAssembly](https://webassembly.org/) in order to use it on the web platform as well as other JavaScript runtimes like Node and Deno.

To achieve this, [some patching](https://github.com/qgis/qgis-js/tree/main/build/vcpkg-ports/qgis-qt6) of QGIS was needed:
- Some small patches to disable certain features that are not (yet) supported
- Tweaking of the CMake structure/scripts
- Some heavy, horizontal patches, for example, the exclusion of the `QgsAuthManager`

Especially the horizontal patches prevent qgis-js from following the QGIS roadmap and keeping it up to date to the latest QGIS version. With this proposal we want to bring those patches upstream to the QGIS project and adopt `wasm32-emscripten` as a supported build target for QGIS.

The primary goal of qgis-js is to port QGIS core to WebAssembly for rendering QGIS projects on the client side, serving as a replacement for QGIS Server. It also includes optional OpenLayers integration for seamless embedding into web applications. While qgis-js's API surface is currently limited, we aim to expand it in the future.

Additionally, compiling QGIS to WebAssembly opens new possibilities, such as porting QGIS processing, bringing PyQGIS to the browser, and building full-screen QGIS apps using Qt Classic or Qt Quick (e.g., porting QField). Although this is outside the scope of qgis-js, upstreaming this work will provide a foundation for developers to explore these opportunities.

## Proposed Solution

The proposed solution involves the following steps:
1. Rebase all patches from qgis-js and integrate them into the main QGIS repository for use by qgis-js and other projects.
2. Update qgis-js to the latest QGIS version and ensure future compatibility and updates.

## Deliverables

1. Integration of qgis-js patches into the main QGIS repository on master.
2. Adopting `wasm32-emscripten` as a build target in CMake/vcpkg
3. Continuous Integration test on Github Actions to ensure Emscripten compatibility in the future.
4. A new major version of qgis-js based on QGIS 3.44

### Affected Files

1. CMake build scripts
   - Turning off certain features when the Emscripten toolchain is detected (e.g. `-DWITH_AUTH:BOOL=FALSE`)
   - We might introduce new feature flags to disable certain features that won't work with Emscripten in CMake
2. Various source files
   - Conditionally exclude certain code blocks with `#ifndef Q_OS_WASM`
      - see [`QtGlobal` docs](https://doc.qt.io/qt-5/qtglobal.html#Q_OS_WASM)
3. New Github Action

## Risks

1. Additional complexity (build system and QGIS sources)

## Performance Implications

Running QGIS in a web browser may have performance implications due to the limitations of the `wasm32-emscripten` target. These implications will need to be evaluated and optimized during the development process but won't affect any other build target of QGIS.

## Backwards Compatibility

This enhancement should not affect the existing build targets.

## Issue Tracking ID(s)

- https://github.com/qgis/qgis-js/issues/39

## WebAssembly demos

- qgis-js
  - https://qgis.github.io/qgis-js
  - https://boardend.github.io/qgis-js-demo
- Geospatial C++ libraries
  - GDAL: https://observablehq.com/@neocartocnrs/gdal3js
  - SpatiaLite: https://jvail.github.io/spl.js/examples/topology.html
- Qt Classic
  - https://ripes.me
- Qt Quick
  - https://www.qt.io/hubfs/Qt%20for%20WebAssembly/slate/app.html
  - https://s3.eu-west-2.amazonaws.com/wasm-qt-examples/last/qtdeclarative/imageelements/imageelements.html
- Python
  - https://starboard.gg/#python
  - https://pyscript.com/@examples
