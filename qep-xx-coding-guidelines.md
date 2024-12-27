# QGIS Enhancement 13: Code style and practice guidelines

## Summary

This QEP documents coding standards and conventions used throughout the QGIS code base. Developers are required to follow these standards when submitting code to QGIS.

Notes:

- Spacing and indentation rules are automatically applied by the QGIS pre-commit hooks, and accordingly are not covered here
- These guidelines describe QGIS specific coding practices only. Following "best practice" for modern c++ coding is assumed, and only referred to here when QGIS coding requires certain variances from this.

## Coding Guidelines

### 1. Naming

1.1. Variable and function naming should be camel case.

  1.1.1. Local variables should have no name prefix (eg do not prefix variable names with ``my``)
  - [Hungarian notation](https://en.m.wikipedia.org/wiki/Hungarian_notation) is NOT used
  1.1.2. Member variables should have a ``m`` prefix (eg ``mLineWidth``)
  1.1.3. Static variables should have a ``s`` prefix (eg ``sMutex``)
  1.1.4. constexpr or static constants should be named in all uppercase, with underscore separators (eg ``DEFAULT_LINE_WIDTH``)
1.2. Classes should be named with a ``Qgs`` prefix, eg ``QgsGeometry``
1.3. Avoid abbreviations in naming (eg use ``maximum`` instead of ``max``, ``width`` instead of ``w``). While
  this can increase the lengths of names it ensures that naming is consistent across the API and
  is clearer for those of a non-native English speaking background. This applies to function names,
  variable names, and argument names.
1.4. Getters and setters should use Qt naming conventions, eg ``setLineWidth()`` for a setter and
  ``lineWidth()`` for the getter.

### Code Documentation

- All public and protected fields must include Doxygen documentation.
- Avoid repetitive documentation. Eg:


```
/**
 * Sets the line width in centimeters.
 * \param width line width in centimeters
 */
```
  
  instead:

```
/**
 * Sets the line \a width in centimeters.
 */
```

  or:

```
/**
 * Sets the line width.
 *
 * \param width line width, specified in centimeters.
 */
```

- Qt style grammar and wording should be used. eg "Returns the line width" instead of "getter for line width", "Sets the line width" instead of "setter for line width", etc.
- Ensure that the first line in the documentation is a concise sentence describing the method or class. This first line is interpreted as a brief summary of the object, and is used in table of contents in the c++ and PyQGIS documentation. The first line must be followed by a blank line, before any detailed explanations are described.
- All methods and classes must have a ``\since QGIS 3.xx`` annotation added, describing the QGIS version when
  that method was added. If the method is to be backported to a stable branch, ensure that the ``\since``
  version correctly describes version at which that method is guaranteed to be accessible. (eg ``\since QGIS 3.34.8``
  instead of ``\since QGIS 3.34``). Methods introduced in the same version as their class do not require an explicit since annotation.
- Avoid re-documenting overridden methods, unless there is **specific** behavior differences in the override
  which added extra useful information over the base virtual method documentation.
- Use ``\see otherMethod()`` to link getters and setters (and other related methods)
- The literal values ``true``, ``false`` and ``nullptr`` should be written in documentation as ``TRUE``, ``FALSE`` and ``NULLPTR`` respectively. (Preprocessing macros correctly replace these with ``true``, ``false`` and ``nullptr`` for the c++ documentation, and ``True``, ``False`` and ``None`` for the PyQgis documentation)



### Language features

- For readability and ease of code review, avoid use of ``auto``. The following exceptions are permitted:

  - ``auto`` should be used for complex types, such as iterators. Eg ``for ( auto it = object.begin(); ...)``
  
- If ``enums`` are to be used outside of a single .h/.cpp file, they should be placed inside the ``Qgis`` namespace.

### Memory safety

- "Factory" methods should return a std::unique_ptr (not a raw pointer), unless Qt parent/child
  ownership is in place
- Methods which take ownership of an object should default to taking a unique_ptr argument, **UNLESS**
  these methods are to be exposed to Python, in which case a raw pointer with the ``SIP_TRANSFER`` annotation
  is required.
