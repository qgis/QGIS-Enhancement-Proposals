# QGIS Enhancement: Add a Processing Algorithm to Convert ASCII Point Cloud Files

**Date** 2025/04/30

**Author** Jean Felder (@ptitjano)

**Contact** jean dot felder at oslandia dot com

**Version** QGIS 3.44

# Summary

Add a new Processing algorithm to QGIS that converts ASCII point cloud files (with custom headers or delimiters) into a format compatible with QGIS point cloud support.
This enhancement improves flexibility for users by allowing them to prepare these files for visualization and analysis within QGIS.

## Current Limitations

`PDAL` is capable of reading ASCII point cloud files via its `readers.text` driver, which allows specification of custom separators, headers, and column mappings. However QGIS does not expose these options at the moment.

Instead, when importing point clouds, QGIS delegates to `untwine` for generating a COPC file, bypassing `readers.text` options entirely. Since `untwine` cannot interpret these custom options (e.g., separator or header definitions), users are limited to using only files with default formatting.

A previous prototype attempted to work around this by creating a temporary ASCII file and feeding it to `untwine`, but this was not a reliable approach (see [PR #61498](https://github.com/qgis/QGIS/pull/61498)).

Based on the PR feedback, we propose to introduce a dedicated processing to prepare text-based point cloud files *before* QGIS attempts to read them.

## Proposed Solution

The proposed solution is divided into two stages:

1. **Add a new command to `pdal_wrench` named `text_conversion`**
   This new command will have the following options:
   - **separator**: Specify the input delimiter (comma, semicolon, tab, space, etc.)
   - **header**: Define which columns correspond to X, Y, Z, intensity, classification, etc.
   - **skip**: Number of lines to ignore at the beginning of the file. [Default: 0]
   - **output**: Output a new ASCII file with default formatting readable by QGIS (using the default `readers.text` configuration)

2. **Add a new Processing algorithm to QGIS**
   This algorithm will allow to:
   - Select the input file
   - Configure parsing options
   - Invoke `pdal_wrench` in the background to perform the conversion
   - Generate the formatted file ready for direct use in QGIS


## Deliverables

- A new command in [pdal_wrench](https://github.com/PDAL/wrench): `text_conversion`
- A new QGIS Processing algorithm: `Convert ASCII Point Cloud File`

### Affected Files

- `src/analysis/processing/pdal/qgsalgorithmpdaltextconversion.cpp`
- `src/analysis/processing/pdal/qgsalgorithmpdaltextconversion.h`

## Risks

Very low. This is a new feature with no changes to existing workflows.

## Performance Implications

N/A

## Backwards Compatibility

N/A

## Issue Tracking ID(s)

- https://github.com/qgis/QGIS/issues/53285
