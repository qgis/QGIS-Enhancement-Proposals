# QGIS Enhancement 425: Stable API policy

## Summary

This QEP documents the policy for maintaining a stable QGIS API and QGIS file formats. Developers must
follow these policies when submitting changes to the QGIS code.

## 1.0 Stable API Policy

The stable API policy guarantees that (with exceptions) plugins and scripts written for a particular
major QGIS release will continue to run without breakage and without requiring change in all future
stable releases in that series. For instance, a plugin written for QGIS 4.2 will run without breakage
in QGIS 4.6, 4.10, and later 4.x releases.
 
- 1.1. The following parts of the QGIS API are subject to the stable API guarantee:
  - 1.1.1 The PyQGIS API
  - 1.1.2 The qgis_process CLI tool
- 1.2. Exemptions: 
  - 1.2.1 Notably, the C++ API is *NOT* subject to the stable API guarantee. Maintainers of third 
    party C++ applications which utilise the QGIS API should understand that the C++ API can be
    freely broken and changed between QGIS stable releases.
  - 1.2.2 Parts of the QGIS codebase which are not exposed to the PyQGIS API (such as the app library and
    the provider implementations) are excluded from the stable API guarantee.
  - 1.2.3 Some parts of the QGIS code are exposed to Python for internal use only, and are not
    subject to the stable API guarantee. These methods must be documented with a
    `\warning This is not considered stable API, and is exposed to Python for internal use only.` annotation
    in the doxygen for the method.
  - 1.2.4 Some parts of the QGIS code are exposed to Python as "tech previews" only, and are not
    subject to the stable API guarantee. These methods must be documented with a
    `\warning This is not considered stable API, and may change in future QGIS releases. It is exposed to the Python bindings as a tech preview only.`
    annotation in the doxygen for the method. (After an appropriate number of stable releases it is
    expected that the API for those classes will be stabilised, and the warning removed.)
- 1.3. Specific notes:
  - 1.3.1 The stable API **includes** argument names for methods, as these can be specified
    via named keywords when using the PyQGIS API. Renaming an existing argument would break
    code using named keywords.
  - 1.3.2 Similarly, argument order is subject to the guarantee and cannot be changed for
    existing methods. New arguments added to an existing method should be optional, with
    appropriate default values set to avoid breakage of existing code.
  - 1.3.3 Return values are subject to the guarantee. It is not permissible to add new
    values to a returned tuple, or change their ordering.
- 1.4. Deprecations
  - 1.4.1 Existing methods and classes can be deprecated by: (all three must be done)
    - 1.4.1.1 Adding a descriptive `\deprecated QGIS x.xx, use yyy instead` annotation to the
      doxygen for the method or class
    - 1.4.1.2 Adding a `Q_DECL_DEPRECATED` prefix before the method declaration
    - 1.4.1.3 Adding a `SIP_DEPRECATED` suffix at the end of the method declaration
  - 1.4.2 When deprecating an existing method, it is the developer's responsibility to ensure
    that all uses of the deprecated method are upgraded to the replacement API. Deprecation
    build warnings will cause CI failures.
    - 1.4.2.1 It is acceptable to use the `Q_NOWARN_DEPRECATED_PUSH` / `Q_NOWARN_DEPRECATED_POP` pairs
      to suppress the deprecation build warnings, **ONLY** for code in methods or classes which
      themselves are deprecated. (I.e. where the code calling the deprecated API will be removed at
      the same time as the deprecated API itself).
  - 1.4.3 Generally, breakage of the deprecated method is NOT permitted until a suitable major release. The
    developer marking a method as deprecated must take care that they do not break any plugins
    or scripts which still use the deprecated API.
  - 1.4.4 Exceptional Breakage: In rare, extreme circumstances, backwards-incompatible API changes
    may be permitted prior to a new major release.
    - 1.4.4.1 This exemption is strictly limited to cases where maintaining backward compatibility
      poses a severe security risk, introduces an insurmountable maintenance burden, or fundamentally
      blocks critical bug fixes.
    - 1.4.4.2 Any such breaking change must be heavily justified, proposed publicly, and explicitly
      approved by a QGIS Enhancement Proposal prior to implementation.
    - 1.4.4.3 Where possible, an accelerated deprecation warning should be provided to users
      for at least one minor release before the breakage occurs.

## 2.0 Stable Document Format Policy

The Stable Document Format Policy relates to file formats specifically designed for use in QGIS, such
as QGS project files or QML styling files. It excludes all external file formats (such as geospatial
data formats), or standards based formats (such as SLD styling files).

- 2.1. The following QGIS generated file formats are subject to the stable document format policy (this
  list is provided as a guide only, it is not exhaustive):
  - 2.1.1 QGS/QGZ project files
  - 2.1.2 QML styling files
  - 2.1.3 QLR layer description files
  - 2.1.4 QMD metadata files
  - 2.1.5 QPT print layout templates
  - 2.1.6 .model3 Processing models
  - 2.1.7 Style database .xml exports
- 2.2. Backward compatibility guarantee: Files created in earlier versions of QGIS must be fully compatible
  with future QGIS releases, with compatibility code in place to ensure that they are interpreted losslessly.
  - 2.2.1. In exceptional circumstances, a major release is permitted to drop compatibility for very old
    objects. For instance, QGIS 3.0 dropped support for the QGIS 1 layer symbology and labeling. This should
    only be done when the burden of maintaining compatibility is extreme and the real-world impact on users
    is minimal. Prior to dropping any compatibility, a QEP must be submitted for wide discussion.
- 2.3. Forward compatibility: no consideration is given to forward compatibility of any QGIS file formats.
  These can be freely changed in any version (whether major, minor or patch), regardless of whether this
  prevents opening newer files in older QGIS releases.
  - 2.3.1 Furthermore, adding compatibility code to newer releases to add partial forward file compatibility
    is discouraged, especially where it bloats the file formats or adds additional code maintenance burdens.
- 2.4. Scope
  - 2.4.1 The stable document format policy has consequences for other parts of QGIS. For instance, QGIS
    expression functions must remain stable and cannot change behavior between QGIS versions, as that may
    cause unintentional breakage of user's projects and documents.
