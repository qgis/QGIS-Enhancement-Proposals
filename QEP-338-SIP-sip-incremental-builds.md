# QGIS Enhancement: SIP Incremental Builds

**Date** 2025-03-25

**Author** David Koňařík (@dvdkon)

**Contact** dvdkon at konarici dot cz

**Version** QGIS 3.46 at the latest

# Summary

QGIS uses [SIP](https://github.com/Python-SIP/sip) to generate its Python
bindings. This means parsing all relevant C++ headers, generating C++ source
files using SIP, and compiling them. The QGIS build system does this whole
process every time any relevant header is changed.

Since SIP 5, the parsing and generation steps have [slowed down
significantly](https://github.com/qgis/QGIS/pull/60291), and compiling the
resulting C++ has never been fast. This has led some developers to stay with
SIP 4, which is, however, unmaintained and doesn't work with PyQt6, so this
will soon stop being an option.

I propose adding the option of building Python bindings incrementally -- per
each header. This should speed up the Python bindings build during development
by orders of magnitude.

## Proposed Solution

SIP 6's `sipbuild` would be used as a Python module in a script that generates
one C++ source file per header file. Currently, `sip-build` generates one file
per class, optionally concatenating them into some fixed number of files, but
the SIP Python API is considerably more flexible.

A new CMake variable would be introduced that switches the bindings build
process from the current `sip-build`-based one to using the new script.

## Deliverables

- Python script that generates Python bindings per-file, rather than
  per-project.
- Integration of this Python script into QGIS' cmake build.

### Affected Files

Mainly `cmake/SIPMacros.cmake` and `python/CMakeLists.txt`, plus the new Python
script.

## Risks

This change will add complexity to the QGIS build system. I propose keeping
this system in parallel to our existing SIP-based process, so that it is only
used by active developers.

In case the SIP Python API changes, the new script will have to be updated to
work. Until then, the non-incremental build process will have to be used.

Given that the files will be generated by the same SIP, the bindings should not
differ from the ones built non-incrementally, but the possibility exists. Such
issues should be caught by CI builds.

## Performance Implications

Runtime code in QGIS should not be affected by this change.
