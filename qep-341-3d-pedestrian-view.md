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

A button will be added to the 3D view toolbar to activate this tool and select where to begin navigation. Once the button is clicked (it stays pressed), a marker will be placed vertically above the center of the 3D view. The user moves the marker to the desired location and confirms with a left mouse click or the Enter key.

Once the mode is activated:

* the camera moves to a human-like height above the terrain at the selected location
* the camera is facing the same direction as before activation
* the movement mode changes (see below)

The user exits this mode by clicking the previous toolbar button (still pressed) or by pressing the *Esc* key or by doing a right click with the mouse.

Optionally, the button can be placed in the Camera menu but since no pop-up or dialog box is displayed, the user will lose the indication that they have switched to that tool and will not know how to exit that tool.

### Movements

Movements can be done using the keyboard or mouse and are free: unconstrained by obstacles or roads.

Forward and backward movement will be controlled by the `up` and `down` keys. Lateral movement will be controlled by the `left` and right keys. Lateral mouse movements will allow you to turn without moving. Vertical mouse movements allow you to look up or down.

Here is the whole key mapping:

* ARROW and WASD : forward/backward/shift left/shift right
  * `Ctrl`: temporary decrease speed
  * `Shift`: temporary increase speed
* `Space`: jump **optional**

Here is the whole mouse mapping:

* mouse move: move look at (up, down, left, right)
* left mouse click: toogle engage/disengage mouse (also bound to QuoteLeft)
* right mouse click: quit mode (also bound to Esc)
* Wheel : permanent speed factor (as in current walk mode)

### Existing navigation modes

The current **Walk** mode will be renamed **Fly** as the pedestrian view is more a real walk mode.

As we will have 3 navigation modes, the 3D UI should display the current mode in use and a way to switch from one to another.

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
