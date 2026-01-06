# QGIS Enhancement: Introduce a data viewer for the model designer

**Date** December 2025

**Author** Valentin Buira

**Contact** valentin at opengis dot ch

**maintainer** @ValentinBuira

**Version** QGIS 4.2

# Summary

Currently, debugging a model inside the model designer is cumbersome and involves multiple back and forths between your model designer and temporary output layer that you have to manage manually in your QGIS project.

We propose to introduce a new data viewer, that would be a dialog specialized to visualize and inspect intermediate step in a model. 

This dialog can then be turned into a dock. It will be composed of components already existing in QGIS, in order to speak the same language as the user, and reduce the overhead of code maintenance.   

This might seem like an easy feature to implement at first glance. What could go wrong it's just a `QDialog` and a `QgsMapCanvas` (*Right ?*). Yet it's also easy to get it wrong, and if we want to build upon the feature it's important to have an agreed upon design. 


## Presentation from the user perspective

### Invoking the data viewer

A new button positioned on the branch of the model will be added. With an icon meaning "Expand" it will invoke the data viewer in a new dialog window.

![Invoking the data viewer1](images/qep406/model_designer_dataviewer_invoke.png)


![Invoking the data viewer2](images/qep406/model_designer_dataviewer_overview.png)

### Anatomy of the data viewer

Once invoked, the data viewer will have three main areas: the toolbar, the map canvas for a visual feedback and an attribute table for the details of data within this model step.

![Anatomy of the data viewer](images/qep406/model_designer_dataviewer_common.png)

# Proposed Solution

We propose to introduce a new specialized dialog, but keep it simple by using existing bricks of QGIS.

## A dockable dialog

Once the dialog is invoked it can then be docked, this way you can stack multiple intermediate steps at the same time, and watch then simultanously. 

![Dockable](images/qep406/model_designer_dataviewer_dockable.png)


You can think of it like a breakpoint in programming except it doesn't stop the execution of the model.


## Update on each run of the model

The data viewer will be updated on each new run of the model, so users will be able to monitor the change in their model at a certain branch. (`QgsProcessingAlgorithmDialogBase::algorithmFinished` signal can be used to achieve this)


> **_NOTE:_**  The data viewer will only be updated when relevant, i.e if the branch or the algorithm watched has not been deleted

## Leverage existing capabilities of QGIS

One of the main strenghts of QGIS is the render engine along and all the utilities around the map canvas. So this QEP proposes to leverage the existings capabilities, both from end user perspective, and code wise.

### Widgets 

At least the following existing code would be used: 

* `QgsMapCanvas`
* `QgsAttributeTableView`

### Toolbars 

As with widgets, the actions in the toolbar at the top of the dialog will try as much as possible to reuse existing methods. For example, on the zoom to selection in the data viewer, the underlying function actually called will be `QgsMapCanvas::zoomToSelected` 
. New functions could be created if the existing one doesn't apply in this case.

<br>

The rest of the code to achieve the QEP is trivial to implement using standard Qt Framework, `QgsDockWidget`, and a new subclasses of `QgsModelDesignerFlatButtonGraphicItem` for the button responsible for opening the dialog on the model canvas.


## Further Considerations/Improvements


### Improve support for inspecting non vector data

While with the current approach, you can view any data supported in QgsMapCanvas. But inspecting the tabular data the attribut table only supports browsing vector data.

However, what does support inspecting other data type in QGIS is the Identify Results Panel. In a future iteration of the data viewer we could also have a Identify widget and they could switch with a stack widget with the attribut table could switch.

we could make with a new convenience custom widget for this similar to what exists for the `QgsAttributeTableView` something like *QgsIdentifyResultsTreeView*. This widget could also be used in other areas of QGIS and external plugins as well. 

![Improve support inspecting non vector data](images/qep406/model_designer_dataviewer_identify.png)


### Dynamic update with selected component

In this proposal, each data viewer dialog/dock is linked to only one output. We could imagine a more dynamic approach and more tied to the model current selection. 

For example:

![Dynamic update with selected component](images/qep406/model_designer_further_dynamic_improvement.png)

## Backwards Compatibility

No breaking change

