.. _qep#[.#]:

========================================================================
QGIS Enhancement 13: Efficient Snapping and Geometry Queries
========================================================================

:Date: 2014/12/18
:Author: Martin Dobias
:Contact: wonder.sk at gmail dot com
:Last Edited: 2015/01/05
:Status:  Final Draft
:Version: QGIS 2.8

Summary
----------

This proposal is meant address several issues with snapping in QGIS:

#. Spacing and indentation rules are automatically applied by the QGIS pre-commit hooks, and accordingly are not covered here
#. These guidelines describe QGIS specific coding practices only. Following "best practice" for modern c++ coding is assumed, and only referred to here when QGIS coding requires certain variances from this.


Coding Guidelines
--------------------

Naming
======

- Variable and function naming should be camel case.

  - Local variables should have no name prefix (eg do not prefix variable names with ``my``)
  - Member variables should have a ``m`` prefix (eg ``mLineWidth``)
  - Static variables should have a ``s`` prefix (eg ``sMutex``)
  - constexpr or static constants should be named in all uppercase, with underscore separators (eg
    ``DEFAULT_LINE_WIDTH``

- Avoid abbreviations in naming (eg use ``maximum`` instead of ``max``, ``width`` instead of ``w``). While
  this can increase the lengths of names it ensures that naming is consistent across the API and
  is clearer for those of a non-native English speaking background. This applies to function names,
  variable names, and argument names.
- Getters and setters should use Qt naming conventions, eg ``setLineWidth()`` for a setter and
  ``lineWidth()`` for the getter.

Code Documentation
==================

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

- All methods should have a ``\since QGIS 3.xx`` annotation added, describing the QGIS version when
  that method was added. If the method is to be backported to a stable branch, ensure that the ``\since``
  version correctly describes version at which that method is guaranteed to be accessible. (eg ``\since QGIS 3.34.8``
  instead of ``\since QGIS 3.34``)
- Avoid re-documenting overridden methods, unless there is **specific** behavior differences in the override
  which added extra useful information over the base virtual method documentation.



Language features
=================

- For readability and ease of code review, avoid use of ``auto``. The following exceptions are permitted:

  - ``auto`` should be used for complex types, such as iterators. Eg ``for ( auto it = object.begin(); ...)``
  
- If ``enums`` are to be used outside of a single .h/.cpp file, they should be placed inside the ``Qgis`` namespace.

Memory safety
=============

- "Factory" methods should return a std::unique_ptr (not a raw pointer), unless Qt parent/child
  ownership is in place
- Methods which take ownership of an object should default to taking a unique_ptr argument, **UNLESS**
  these methods are to be exposed to Python, in which case a raw pointer with the ``SIP_TRANSFER`` annotation
  is required.
