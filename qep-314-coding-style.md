# QGIS Enhancement 314: Code style and practice guidelines

## Summary

This QEP documents coding standards and conventions used throughout the QGIS code base. Developers are required to follow these standards when submitting code to QGIS.

Notes:

- Spacing and indentation rules are automatically applied by the QGIS pre-commit hooks, and accordingly are not covered here
- These guidelines describe QGIS specific coding practices only. Following "best practice" for modern c++ coding is assumed, and only referred to here when QGIS coding requires certain variances from this.

## Coding Guidelines

### 1. Naming

- 1.1. Variable and function naming should be camel case. Generally Qt's approach to capitalising only the first letter in acronyms is used (eg ``QgsGpsDetector``, not ``QgsGPSDetector``), with the exception of ``3D`` (eg ``Qgs3DMapCanvas``).

  - 1.1.1. Local variables should have no name prefix (eg do not prefix variable names with ``my``)
  - 1.1.2. [Hungarian notation](https://en.m.wikipedia.org/wiki/Hungarian_notation) is NOT used, except in some exceptional circumstances (eg variable names used with the gdal c API)
  - 1.1.3. Member variables should have a ``m`` prefix (eg ``mLineWidth``)
  - 1.1.4. Static variables should have a ``s`` prefix (eg ``sMutex``)
  - 1.1.5. constexpr or static constants should be named in all uppercase, with underscore separators (eg ``DEFAULT_LINE_WIDTH``)
- 1.2. Classes should be named with a ``Qgs`` prefix, eg ``QgsGeometry``
- 1.3. Avoid abbreviations in naming (eg use ``maximum`` instead of ``max``, ``width`` instead of ``w``). While
  this can increase the lengths of names it ensures that naming is consistent across the API and
  is clearer for those of a non-native English speaking background. This applies to function names,
  variable names, and argument names.
- 1.4. Getters and setters should use Qt naming conventions, eg ``setLineWidth()`` for a setter and
  ``lineWidth()`` for the getter.

### 2. Code Documentation

- 2.1. All public and protected fields must include Doxygen documentation.
- 2.2. Avoid repetitive documentation. Eg:


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

- 2.3. Qt style grammar and wording should be used. eg "Returns the line width" instead of "getter for line width", "Sets the line width" instead of "setter for line width", etc.
  - 2.3.1. Use full sentences with correct punctuation (capitalisation and full stops) to ensure the documentation conveys professionality.
- 2.4. Ensure that the first line in the documentation is a concise sentence describing the method or class. This first line is interpreted as a brief summary of the object, and is used in table of contents in the c++ and PyQGIS documentation. The first line must be followed by a blank line, before any detailed explanations are described.
- 2.5. All methods and classes must have a ``\since QGIS 3.xx`` annotation added, describing the QGIS version when
  that method was added. If the method is to be backported to a stable branch, ensure that the ``\since``
  version correctly describes version at which that method is guaranteed to be accessible. (eg ``\since QGIS 3.34.8``
  instead of ``\since QGIS 3.34``). Methods introduced in the same version as their class do not require an explicit since annotation.
- 2.6. Avoid re-documenting overridden methods, unless there is **specific** behavior differences in the override
  which added extra useful information over the base virtual method documentation.
- 2.7. Use ``\see otherMethod()`` to link getters and setters (and other related methods)
- 2.8. The literal values ``true``, ``false`` and ``nullptr`` should be written in documentation as ``TRUE``, ``FALSE`` and ``NULLPTR`` respectively. (Preprocessing macros correctly replace these with ``true``, ``false`` and ``nullptr`` for the c++ documentation, and ``True``, ``False`` and ``None`` for the PyQGIS documentation)
- 2.9. Include explicit details of memory ownership in the documentation. Eg "ownership of the symbol is transferred to the renderer", "caller takes ownership of the returned object".


### 3. Language features

- 3.1. For readability and ease of code review, avoid use of ``auto``. The following exceptions are permitted:

  - 3.1.1. ``auto`` should be used for complex types, such as iterators. Eg ``for ( auto it = object.begin(); ...)``
  - 3.1.2. ``auto`` may be used for variable types if the type is explicit during variable initialization. Eg

```
// allowed, as the QgsPoint type is explicit during initialization:
auto pointObject = QgsPoint( 3, 4 );
// allowed, as the std::unique_ptr< QgsPoint >, std::shared_ptr< QgsPoint > types are explicit during initialization:
auto pointUniquePointer = std::make_unique< QgsPoint >( 3, 4 );
auto pointSharedPointer = std::make_shared< QgsPoint >( 3, 4 );

// NOT allowed, the argument types for the std::tuple are not explicit:
auto myTuple = std::make_tuple( 0, 5 );
```

- 3.2. If ``enums`` are to be used outside of a single .h/.cpp file (or there is a reasonable chance that they will be in future!), they should be placed inside the ``Qgis`` namespace.

- 3.3. Checking if a pointer is null should be done with ``if ( !ptr )`` or ``if ( ptr )`` alone, omitting explicit comparison with the ``nullptr`` constant.
  
- 3.4. Always use ``std::as_const`` to wrap the iterated container when iterating over non-const Qt containers (ie QList, QVector, QHash, ...). E.g.

```
QList< int > someList;
for ( int value : std::as_const( someList ))
{
 ...
}
```

- 3.5. Use ``QStringLiteral`` for untranslated literals, and ``QLatin1String`` for string comparisons. E.g.

```
const QString s = QStringLiteral( "my string" );
if ( s == QLatin1String( "another string" ) )
    ...
```

- 3.6. Don't use ``qDebug()``, ``qWarn()`` or other Qt debug print functions. Instead use ``QgsDebugError`` for unexcepted error logging only, or ``QgsDebugMsgLevel`` (with a level of 2 or higher, depending on how "noisy" the logging will be) for debug outputs which occur in normal operations.
- 3.7. Member variables should normally be in the private section and made available via getters and setters. This ensures full compatibility with the PyQGIS bindings and allows refactoring in future without API breakage.
- 3.8. Avoid use of Qt "auto connect slots" (i.e. those named ``void on_mSpinBox_valueChanged``). Auto connect slots are fragile and prone to breakage without warning if UI files are refactored.

### 4. Memory safety

- 4.1. "Factory" methods should return a std::unique_ptr (not a raw pointer), unless Qt parent/child
  ownership is in place
- 4.2. Methods which take ownership of an object should default to taking a unique_ptr argument, **UNLESS**
  these methods are to be exposed to Python, in which case a raw pointer with the ``SIP_TRANSFER`` annotation
  is required.

### 5. File structure 

- 5.1. Includes should be at the top of the .h or .cpp files. Group QGIS includes together, and where possible, include QGIS headers before Qt or other external library headers.
- 5.2. When including Qt headers, use ``#include <QString>`` instead of ``#include qstring.h``.
- 5.3. Avoid module wide Qt imports, i.e. don't ``#include <QtGui>``.
