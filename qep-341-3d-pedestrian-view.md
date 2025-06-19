# QGIS Enhancement: Pedestrian walk mode in the 3D Map View

**Date** 2025/06/18

**Author** Benoit De Mezzo ([@benoitdm-oslandia](https://github.com/benoitdm-oslandia)), Jean Felder ([@ptitjano](https://github.com/ptitjano))

**Contact** benoit dot de dot mezzo at oslandia dot com, jean dot felder at oslandia dot com

**Version** QGIS 4.X

## Summary

This QGIS enhancement adds a human-level navigation mode to the 3D view (like Google Street View). It complements the two existing navigation modes.

This mode can be activated by selecting a specific point or by default from the center of the map. Once this navigation mode is activated, the camera's elevation is locked at 1.80 meters above the ground. This height can be configured in the 3D view configuration dialog.

When this mode is activated, a small inset with the 2D view is superimposed on the 3D view.

## Proposed Solution

### Enabling and disabling this tool

A button will be added to the 3D view toolbar to activate this tool and select where to begin navigation. Once the button is clicked (it stays pressed), a marker will be placed vertically above the center of the 3D view. The user moves the marker to the desired location and confirms with a left mouse click or the Enter key.

Once the mode is activated:

* the camera moves to a human-like height above the terrain at the selected location
* the camera is facing the same direction as before activation
* the 2D map inset is displayed (see below)
* the movement mode changes (see below)

The user exits this mode by clicking the previous toolbar button (still pressed) or by pressing the *Esc* key or by doing a right click with the mouse.

Optionally, the button can be placed in the Camera menu but since no pop-up or dialog box is displayed, the user will lose the indication that they have switched to that tool and will not know how to exit that tool.

### Movements

Movements can be done using the keyboard or mouse and are free: unconstrained by obstacles or roads.

Forward and backward movement will be controlled by the up and down keys (and mouse wheel). Lateral movement will be controlled by the left and right keys (and lateral movement of the mouse wheel). Lateral mouse movements will allow you to turn without moving. Vertical mouse movements allow you to look up or down.

In order to match existing, binding to W/A/S/D keys (for forward, left, backward and right) will also be added.

Movement speed can be changed by holding `Ctrl` (decrease speed) or `Shift` (increase speed) keys.

### 2D view inset

When this mode is enabled, an inset with the 2D view is displayed as an overlay to help the user navigation. It is centered on the user's current X/Y position, and its extent is adjusted based on the camera's tilt (the larger the far plane, the larger the extent).

This inset could also be a standalone feature enabled from the 3D view settings ("Navigation Synchronization").

As in Google Street View, the position and size of this inset would be fixed.

See <https://github.com/qgis/QGIS-Enhancement-Proposals/pull/341#discussion_r2156539364>.

### Existing navigation modes

The current **Walk** mode should be renamed **Fly** as the pedestrian view is more a real walk mode.

As we will have 3 navigation modes the 3D UI should display the current mode in use and a way to switch from one to another.

## Deliverables

* 2D view inset class
* new maptool class
* new toolbar button image

### Example(s)

See associated [PR](https://github.com/qgis/QGIS-Enhancement-Proposals/pull/341).

### Affected Files

*(optional)*

## Risks

None

## Performance Implications

None

## Further Considerations/Improvements

* identification

## Backwards Compatibility

Yes
