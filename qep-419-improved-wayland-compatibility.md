# QGIS Enhancement: Improve QGIS Wayland compatibility

**Date** 2026-04-26

**Author** Nyall Dawson (@nyalldawson)

**Contact** nyall dot dawson at gmail dot com

**Version** QGIS 4.2 or later, depending on upstream considerations

# Summary

Over the last couple of years, most Linux distributions and desktop environments have been transitioning
from the legacy X11 display server to the Wayland standard. The transition has reached a stage where
many environments are now completely dropping support for the X11 server.

This presents a challenge to Linux QGIS users, as the current QGIS versions have limitations and bugs
when run in a Wayland environment.

This QEP aims to address these limitations in the "best possible" way, given the (many!) constraints of
Wayland.

Unfortunately, the Wayland maintainers are strongly opinionated have very strict ideas about what a
modern application should be allowed to do, and their vision does not fit well with the
needs of QGIS. While it will not be possible to completely support the full capabilities and user workflows
that are possible in X11, there are opportunities to fix some of the current shortcomings.

# Issues to be addressed

_**Note that Wayland is a standard only, and support for the standard varies between desktop environments
(eg Gnome vs KDE). Limitations in the Wayland implementation for a particular environment may mean that
the fixes described below DO NOT APPLY for that environment. This is out of the control of QGIS developers.**_

## Broken color picker

QGIS allows for users to easily sample colors from anywhere on the screen, and apply these colors to
their map symbology. This functionality is currently completely broken under Wayland, as it relies on
X11 functionality to be able to capture the entire screen contents.

In order to fix this, there are a few possible approaches:

1. Limit the color picking to areas of the screen which Wayland permits us to capture (ie the current 
   active QGIS window only)
2. Potentially we could rely on the freedesktop "Screenshot" portal to first capture the entire screen, and then 
   somehow sample the colors from that captured image. This would have the downside that for every color sample
   done, the user would have to agree to a forced "allow application to capture the screen" prompt. 
3. We could port to the freedesktop "PickColor" portal, which is intended for use cases like ours. Unfortunately 
   this still has limitations, such as no live sampling of colors as the mouse moves, and we would have to
   remove QGIS' ability to sample an average color over a range of pixels (which is not supported by the Freedesktop
   portal).

We will test each of these approaches and ultimately implement the "best" solution possible (with the understanding
that no solutions will be a perfect match for the X11 functionality).

To facilitate this, the screenshot or color sampling functionality will be moved to a new QgsNative method, allowing
us to leave the more full-featured implementations available on non-Wayland platforms.

## Broken mouse warping

QGIS 3D maps allow for a navigation mode which mimics that of first-person video games. This relies on the ability
to "warp" the mouse pointer (programmatically moving it to a different location on screen), so that the user can
continually move the mouse to rotate the view indefinitely. Without pointer warping, the mouse will hit the edge of the
screen and further movement is ignored.

While Wayland has previously prohibited cursor warping (for "security" reasons), it recently introduced a "Pointer warp
protocol". Support for this protocol was added in Qt 6.11.

We will assess whether Qt 6.11's support is sufficient to fix the 3D navigation handling on Wayland for QGIS, or whether
we will need to implement direct support for the Pointer Warp Protocol.

If required, support for pointer warping will be moved to a new QgsNative method, allowing
us to leave the standard Qt cross-platform approach for non-Wayland platforms.

## Broken window placement

QGIS is an inherently multi-window application, where a single-window mode would be severely limited and a massive
regression for QGIS users.

Unfortunately, multi-window applications have been an afterthought for the Wayland maintainers. After years of discussion
they have finally decided to compromise and accept that a multi-window application is still a valid requirement in 2026!
This saw the introduction of the Wayland "session management" protocol.

For QGIS users, this means that running under Wayland environments results in random window placement. Eg a layout designer
window will randomly open in different locations, or on different screens, with no predictable pattern for users. Window
placement is NOT remembered, so users continually have to re-shuffle their windows for their preferred arrangement.

The Wayland Session Management protocol should help alleviate this pain, by providing a way for applications to tag
windows so that their window manager can correctly restore their previous geometry.

Support was added for this protocol in Qt 6.11, however, it requires changes in the application to work.

We will assess Qt 6.11's support, and make the necessary changes in QGIS to handle this protocol.

## Crashes in docking / floating windows

It's possible to crash QGIS via certain combinations of undocking and stacking panels or toolbars (under KDE).
We will assess whether there is anything QGIS can do to prevent this, or if not, provide detailed reports to the upstream
Qt project to try to get these issues fixed.

## Risks

None

## Performance Implications

None, theoretically Wayland is "better" and QGIS/QGIS 3D may run better under these environments. Emphasis on "may" ;) 

## Backwards Compatibility

If any fixes implemented as part of this work are self-contained, we will assess whether they are suitable for
backporting.
