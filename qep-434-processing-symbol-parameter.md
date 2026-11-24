# QGIS Enhancement 434: A "Symbol" parameter for Processing algorithms

## Summary

This proposal describes a new "symbol" parameter type for use in Processing algorithms.

It would allow creation of algorithms that work with symbol objects, such as assigning a symbol
to a layer.

This symbol parameter type will predominantly be useful for modeler-specific algorithms which
automate creation of styles for output layers.

## Proposed Changes

### Introduction of QgsProcessingParameterSymbol

A new parameter type will be introduced to the Processing API, `QgsProcessingParameterSymbol`.

The API will follow other similar parameter types (eg `QgsProcessingParameterColor`). Specifically,
the header will be:

```
/**
 * \class QgsProcessingParameterSymbol
 * \ingroup core
 * \brief A symbol parameter for processing algorithms.
 *
 * QgsProcessingParameterSymbol should be evaluated by calling QgsProcessingAlgorithm::parameterAsSymbol().
 *
 * \since QGIS 4.6
 */
class CORE_EXPORT QgsProcessingParameterSymbol : public QgsProcessingParameterDefinition
{
  public:
    /**
     * Constructor for QgsProcessingParameterSymbol.
     *
     * The \a symbolType argument specifies the type of symbol allowed.
     */
    QgsProcessingParameterSymbol( const QString &name, const QString &description = QString(), Qgis::SymbolType symbolType, const QVariant &defaultValue = QVariant(), bool optional = false );

    /**
     * Returns the type name for the parameter class.
     */
    static QString typeName() { return u"symbol"_s; }

    ... standard overrides ...

    /**
     * Returns the symbol type.
     *
     * \see setSymbolType()
     */
    Qgis::SymbolType symbolType() const;

    /**
     * Sets the symbol \a type.
     *
     * \see symbolType()
     */
    void setSymbolType( Qgis::SymbolType type );

  private:
    Qgis::SymbolType mSymbolType;
};

```

A corresponding `QgsProcessingParameterTypeSymbol` implementation of `QgsProcessingParameterType` will be
created and added to the parameter type registry.

The widget wrapper will create a `QgsSymbolButton` widget, ready for users to configure the symbol parameter
using the standard QGIS symbol designer widget.

### Representation of QgsProcessingParameterTypeSymbol values

Since the Processing parameter API is designed around storing and retrieving parameter values using
QVariants, we cannot directly store QgsSymbol objects as parameter values.

Instead, the new parameter will accept two different variant value forms:

1. A simple string value, containing the raw QGIS XML representation of the symbol (i.e the same symbol XML as is written
   to QGIS project or QML files). 
2. A variant map, following the form:
```
{
  'format': 'symbol format string',
  'symbol': variant map or value containing actual symbol data
}
```

When the second form is used, the existing `QgsAbstractSymbolConverter` implementations will be used
to interpret the symbol data into an actual `QgsSymbol` object. This will allow specifying the symbol data
in a range of convertible forms (which are already implemented in QGIS), such as SLD, Esri REST, OGR style
strings, or MapBox GL styles.

In order to facilitate this, a new `QgsSymbolConverterRegistry` will be created as a central place to store
and retrieve the corresponding `QgsAbstractSymbolConverter` implementations.

### Algorithm support

Initially, the new symbol parameter type will be accompanied by a single algorithm: `Set layer symbol`. This
algorithm will have inputs for a vector layer, and three (optional) symbol parameter inputs (for marker symbol, line
symbol and polygon symbols.)

When the algorithm is executed, the layer's renderer will be set to a single symbol renderer for the symbol
which matches the layer's geometry type (eg the marker symbol for a point layer).

While out of scope for the initial project, in future more algorithms are expected to be introduced which utilise
the new symbol parameter type, such as:

- Set symbol for categorized class
- Set symbol for graduated class
- Set symbol color or size (allowing a model to override the color or size of a symbol parameter prior to setting it as a layer's style)
- Get symbol from style library
- ...

## Risks

None identified at this stage.

## Performance Implications

None

## Further Considerations/Improvements

None

## Backwards Compatibility

Not possible -- the new parameter will only be available in QGIS 4.6 or later

## Issue Tracking ID(s)

None

