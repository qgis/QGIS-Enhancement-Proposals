# QGIS Enhancement: Pedestrian walk mode in the 3D Map View

**Date** 2025/06/18

**Author** Benoit De Mezzo ([@benoitdm-oslandia](https://github.com/benoitdm-oslandia)), Jean Felder ([@ptitjano](https://github.com/ptitjano))

**Contact** benoit dot de dot mezzo at oslandia dot com, jean dot felder at oslandia dot com

**Version** QGIS 4.X

## Summary

This QGIS enhancement adds a human-level navigation mode to the 3D view (like Google Street View). It complements the two existing navigation modes.

This mode can be activated by selecting a specific point or by default from the center of the map. Once this navigation mode is activated, the camera's elevation is locked at 1.80 meters above the ground. This height can be configured in the 3D view configuration dialog.

## Proposed Solution

### Enabling and disabling this tool

A drop-down menu (with the 3 navigation modes) in the "camera" tool in the toolbar to easy the switching from one mode to another.

Once this mode is activated, the user will select where to begin navigation: a marker will be placed vertically above the center of the 3D view. The user moves the marker to the desired location and confirms with a left mouse click or the Enter key.

Once the user click to select the starting point:

* the camera moves to a human-like height above the terrain at the selected location
* the camera is facing the same direction as before activation
* the movement mode changes (see below)

The user exits this mode by switching to another navigation mode or by pressing the *Esc* key.

### Movements

Movements can be done using the keyboard or mouse and are free: unconstrained by obstacles or roads. The user moves are restricted in the current extent.

Forward and backward movement will be controlled by the `up` and `down` keys. Lateral movement will be controlled by the `left` and right keys. Lateral mouse movements will allow you to turn without moving. Vertical mouse movements allow you to look up or down.

Here is the whole key mapping:

* ARROW and WASD : forward/backward/shift left/shift right
  * `Ctrl`: temporary decrease speed
  * `Shift`: temporary increase speed

Here is the whole mouse mapping:

* mouse move: move look at (up, down, left, right)
* Wheel : permanent speed factor (as in current walk mode)

### Existing navigation modes

The current **Walk** mode will be renamed **Fly** as the pedestrian view is more a real walk mode.

As we will have 3 navigation modes, the 3D UI should display the current mode in use and a way to switch from one to another.

### Elevation

Elevation will be computed with the `heightAt` function. As the actual implementation is to coarse, we are working on a pull request in order to improve it.

When the user moves in a no-data area the previous valid elevation will be used.

### Integration

The pedestrian view map tool will handle the location picking stage (with a rubberband) in its dedicated class file (something like `QgsWalkModeMapTool`) and this map tool will call API functions at the "app" level to activate the walk mode at the user specified location (and set it back at the previous mode when it quits).

## Deliverables

* new maptool class
* new toolbar button image

### Example(s)

See associated [PR](https://github.com/qgis/QGIS-Enhancement-Proposals/pull/341).

### Affected Files

* QgsCameraController
* map3dconfigwidget.ui
* Qgs3DMapConfigWidget

## Risks

None

## Performance Implications

None

## Further Considerations/Improvements

* identification without changing of maptool
* add a configuration panel to set key/mouse bindings according to user needs

## Backwards Compatibility

Yes
