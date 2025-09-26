# QGIS Enhancement: Port the model designer canvas from the Graphics View Framework to Qt Quick 

**Date** September 2025

**Author** Valentin Buira (@ValentinBuira)

**Contact** valentin at opengis dot ch, mathieu at opengis dot ch

**Version** QGIS 4.2

# Summary

With QGIS 4 around the corner and the transition to Qt 6. We could leverage the ability of Qt Quick library to create fluid and dynamic user interfaces, to prepare the model designer for the years to come. 

Benefice of using Qt Quick:
* Allow using modern control directly on the model canvas unlike in the graphics view where inserting a QWidgets in the scene is so performance heavy it’s actually frown upon to use (https://www.qt.io/blog/2017/01/19/should-you-be-using-qgraphicsview)
* Speed up development for interactive feature, decrease code debt with a better separation of concerns. With Qt Quick we once we have set the model we don’t need to worry about how the data will be updated to the view, effectively scrapping the need to manually call repaint.
* Would allow in the future to have features like: 
    * a live preview 
    * Dynamic visual feedback to the users

QGIS already built with Qt Quick thanks to the QML widget in form attribut. And QML is already a proven library for UI that have been tested for multiple years by mobile oriented project like QField and Mergin Maps.

*If accepted it would be funded by the QGIS Anwendergruppe Schweiz*

### What are we talking about ?

This QEP is not about porting the whole model designer to QML but only the canvas implemented today as a QGraphicsView, QGraphicsScene and QGraphicsItem.

And to port the interactions around it. The rest of the model designer for example the dialog when editing an algorithm or an input remains unchanged.

![alt text](<./images/qep346/overview what to port.png>)

## Proposed Solution


### The entry point of QML : A QQuickWidget

The QQuickWidget is a regular widget but display a QQuickWindow to display a QML scene. The QQuickWidget will replace the current QgraphicsView and hold much less responsibility as other will take over this role.

For example:

![alt text](<./images/qep346/qquickwidget-qgis.png>)

  *Working QQuickWidget with a single rectangle in QML. One tab for qml one tab for QGraphicsView*

This is also the widget that will quick start the scene in qml, acting as an entry point for the rest of the code and the QEP.

### the model canvas(aka the scene)

One step further under the view, the canvas will be the entry point for every items, as well as hold property on scale, transform etc...

This is a bit similar to the map canvas implemented in qt quick few years ago

`ModelCanvas.qml`

```
Item{
   id:modelCanvasRoot

  Background {
    anchors.fill: parent
  }

  MouseArea {
    onClick
    onWheel
  }

  Repeater

  etc...
}
```

### The controller 

A controller object would make the bridge between the model designer dialog and the opposite. This role today is shared between the QgsGraphicsView/Scene depends on the functions.


```
  graphController.onPaste {
    console.log("Paste items")
  }
```

```
MouseArea {
    id: area
}

  Connections {
    target: area
    function onClicked(mouse) { controler.edit(mouse) }
}
```

### The model, the view and the delegate

Qt quick have a declarative UI that is automatically updated as the underlying model change. Which would avoid a lot of manual redraw a lot of use after free error and where the a QgraphicsItems has been deleted in the last repaint.

```
Repeater {
    id: nodesRepeater
    model: canvasController.nodeModel
    delegate: NodeComponentItem
```

The Repeater, is the view type is used to create a large number of similar items. 

the canvasController.nodeModel would be an QAbstractListModel around what is today the list of child algorithm `QMap<QString, QgsProcessingModelChildAlgorithm> QgsProcessingModelAlgorithm::mChildAlgorithms`

The delegate would be our today `QgsModelComponentGraphicItem`

### The delegate node item 

The delegate is responsible to render itself and provide user input capabilities. Every node in the model should share a common part which is today the base class `QgsModelComponentGraphicItem` and specialized part for today subclasses of `QgsModelComponentGraphicItem` (an input, algorithm, and output). The specialize part would be loaded using a `Loader` QML Type.


``` 
Component {
  id: NodeComponentItem
  pos: 123.4 // properties common to every stored in .model3
  height : 20
  Loader {
    id: NodeTypeLoader
    // Load the good qml for each node type
    idName: "AlgorithmName" + node.id
    sourceComponent: outputComponentItem.qml || childAlgComponentItem.qml
  }
}
```

### Adopt the same approach for every item type in the modeler

The same proven model/view/delegate approach will be used for others items in the model designer. e.g: links, comments, group box etc...


### The tools

The modeler have now a number of tools, both accessible with the UI or via shortcut (e.g the pan tool).

 A tool is generally composed of at press event, release event, and/or a mouse event. And on top of the tool a rubberBand for the UI 

All these transpose as well in QML. through MouseArea and creating a highlight item associate as the rubber band.

### The export the model to svg/import/png buttons

One of the nice feature of the model designer is to be able to export your model for your research methodology. 

It is one of the trade of to use Qt quick. The export to pdf and svg are more complex. Since Qt Quick doesn't use QPainter but OpenGL scenegraph for rendering. We would have to write our own export for pdf and svg.

### A code base shared with the layout

The layout and the model designer use both the same framework with some components duplicate between the two and some others components shared.

One important feature that is shared with the layout is the mouse handle interaction to move a component around, resize etc... This is implemented through QgsGraphicsViewMouseHandles (and subclass QgsModelViewMouseHandles)

When this QEP will be implemented the code duplication will effectively end and both part would go their own way.

The QgsModelViewMouseHandles will be ported to QML  

### Along side with the current QGraphicsView implementation

Given the complexity and risks involved. We will keep the current QGraphicsView implementation and the newer Qt Quick implementation co-exists together for as long as needed. 

We would drop the old implementation of the model canvas only once we meet two conditions: reach feature parity, and the usability from the user perspective is the same or better than with the current implementation.

### backward compatibility of .model3 file  

The new implementation will share the same `.model3` file used in QGIS modeler. the loaded .model3 using `QgsProcessingModelAlgorithm::loadVariant` will just be exposed differently to the scene but it would happen at runtime.

### Note on naming

There are too much things named model since we are not going to rename the Qt terminology, we will try to avoid the name `model` alone in Qgs classes. 

## Deliverables


### Example(s)

![alt text](<./images/qep346/qquickwidget-qgis.png>)


![alt text](<./images/qep346/node-editor-factice.png>)

*Example of a node editor in QML based on GraphFlow if it where in QGIS. This will not be the final looks of the PR, just as a exemple of another working node editor*

### Affected Files

*(optional)*

### Mapping classes 1-to-1

This list is both to not miss anything and to track our progress.

| Current class  | in Qml |
| ------------- | ------------- |
| QgsModelViewTool  |  &nbsp;  |
| QgsModelViewToolLink  |   &nbsp;     |  
| QgsModelViewToolPan  |   &nbsp;     |  
| QgsModelViewToolSelect  |   &nbsp;     |  
| QgsModelViewToolTemporaryKeyPan  |  &nbsp;      |  
| QgsModelViewToolTemporaryKeyZoom  |  &nbsp;      |  
| QgsModelViewToolTemporaryMousePan  |  &nbsp;        |  
| QgsModelViewToolZoom  |    &nbsp;    |  
| QgsModelArrowItem  |    &nbsp;    |  
| QgsModelComponentGraphicItem and child  |    &nbsp;    |  
| QgsModelDesignerFlatButtonGraphicItem  |    &nbsp;    |  
| QgsModelViewRubberBand  |    &nbsp;    |  
| QgsModelViewMouseEvent  |    &nbsp;    |  
| QgsGraphicsViewMouseHandles  |    &nbsp;    |  
| QgsModelViewSnapMarker ( not used TBC ?  )    |    &nbsp;    |  
| QgsModelGraphicsScene  |    &nbsp;    |  
| QgsModelGraphicsView  |    &nbsp;    |  

## Risks

The biggest risk would be to not reach feature parity.

## Performance Implications

Should improve both in terme of speed and of feeling of "snapiness"

## Further Considerations/Improvements

See the benefice on summary

## Backwards Compatibility

Only unstable API should be touched
