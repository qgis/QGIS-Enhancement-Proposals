# QGIS Enhancement: Attribute Form QML Widget Editing Capabilities

**Date** 2026/02/01

**Author** Mathieu Pellerin (@nirvn)

**Contact** mathieu@opengis.ch

**Version** QGIS 4.2

# Summary

Since 2018, users have gained the ability to add QML widgets to enhance their
attribute forms. These widgets have so far been read-only and used to display
feature attributes through QML scenes utilizing QML items such as graphs.

While this is a great addition when using feature forms to read features, the
QML widgets could be even more useful if they would allow for feature
attribute editing. 

## Proposed Solution and Benefits

This QEP proposes the introduction of a new object accessible within the QML
widget’s scene with invokable functions to act as a bridge between the scene
within QML widget and its attribute form. The newly introduced object – injected
into the scene’s root context - will allow for accessing the attribute form
context and enable attribute editing of the attribute form’s current feature.

This additional ability for QML widgets can unlock a world of new possibilities
by allowing for QML components to drive the creation of new user interface to
manipulate feature attributes.

### API Considerations

For QML widgets to be able to edit the attribute form’s current feature, the
`QgsWidgetWrapper` class will need to be tweaked to add a signal to handle
attribute value changes. 

As for the QML widget itself, a new `QgsAttributeFormQmlWidgetBridge` object
will be injected into the root context of the QML scene that will offer a few
invokable functions and a context property to drive attribute editing. 

```
class QgsAttributeFormQmlWidgetBridge : public QObject
{
    Q_OBJECT

    Q_PROPERTY( QgsAttributeFormContext context READ context NOTIFY contextChanged )

  public:
    Form() = default;
    ~Form() = default;

    QgsAttributeFormContext context() const;
    void setContext( const QgsAttributeFormContext &context );

    Q_INVOKABLE setAttribute( const QString &name, const QVariant &value );
}
```

The object itself will be able to be expanded in the future to add more
functionalities if need be.

Note that the `QgsAttributeFormContext` class is already a `Q_GADGET`, which
means the only work required on that front will be to expose some of its
properties as `Q_PROPERTY` for QML widgets to be able to receive contextual
awareness of its current mode, the current feature, the parent feature, etc.

## Deliverables

A new  `QgsAttributeFormQmlWidgetBridge` class which will act as the bridge
between the scene within the QML widget and its attribute form.

### Affected Files

- qgsattributeform.cpp
- qgsattributeformcontext.h
- qgswidgetwrapper.cpp
- qgswidgetwrapper.h
- qgsqmlwidgetwrapper.cpp
- qgsqmlwidgetwrapper.h

## Risks

None
