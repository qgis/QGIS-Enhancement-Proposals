# QGIS Enhancement: Restore the print layout HTML item for QGIS 4

**Date** 2026-04-10

**Author** Nyall Dawson (@nyalldawson)

**Contact** nyall dot dawson at gmail dot com

**Version** QGIS 4.4

# Summary

An ongoing pain point for the QGIS project was QGIS 3.x's reliance on the Qt Webkit library. This library
has long been deprecated by the Qt maintainers, and was finally completely removed in Qt 6.

A direct consequence of this is that functionality which relied on Qt Webkit has been dropped in QGIS 4.0. This
includes:

- The print layout HTML item
- The HTML annotation item

This project will resurrect the print layout HTML item. **_The underlying low-level changes will also facilitate 
a potential replacement for the HTML annotation item, but this is out-of-scope for the current project._**

Note that the functionality required for the QGIS HTML item is also required for some third-party QGIS plugins,
such as the DataPlotly plugin. These plugins are currently broken on QGIS 4.x as a result.

## Background

The upstream Qt project has forced applications to move from the old Webkit classes to newer, maintained "Qt Webengine"
classes. Unfortunately these are not drop-in replacements, and require significant reworking on the application
side to handle. Furthermore, the Webengine classes have NO support for some critical functionality required
by QGIS users, specifically, the ability to render a HTML page as a vector graphic. The Webengine classes only
support rendering of low-dpi rasterised versions of HTML pages.

This is a significant shortcoming, as using a low-resolution rasterised image in QGIS print layouts results in
a much inferior output product for QGIS users.

As part of the lead-up to the QGIS 4.0 release, the QGIS project directly funded part of the work to moving 
to Qt Webengine. This involved a research project to determine possible solutions.

The best solution identified involves:

1. Using Qt webengine's ability to export a page to PDF, which DOES export a vectorised version of the webpage
2. Using an external library (PDF4QT) to render that PDF as a vector graphic within the print layout HTML item

This isn't a perfect solution. It's a complicated setup, and unfortunately the PDF4Qt library force converts text
from the PDF to paths (instead of text objects). But, as determined by the research project, it's the best
solution we have available.

The research project led to development of some classes in QGIS for rendering PDF content as vectors to a QPainter.
It did not cover transitioning the HTML layout items to use the new approach, and accordingly, for QGIS 4.0 the
HTML layout item was disabled.

## Remaining work to be done

Following up from the previous work, the following tasks remain. These will be addressed during this project.

1. Implement API for exporting HTML content from the webengine classes to PDF asynchronously. For thread-safe
   layout rendering, we need to be able to render the HTML content without triggering any event loop on the
   main thread. Since the webengine classes require an event loop to export to PDF, a similar approach to
   the mature QgsBlockingNetworkRequest class will be adapted. This has proven to be a stable, safe way
   to handle a situation like this.
2. Add high-level APIs for rendering HTML to a QPainter surface asynchronously. These will be used by QGIS' 
   HTML layout item, but also could be of use for third party plugins (such as the DataPlotly plugin) which 
   also depend on HTML to QPainter rendering.
3. Port the layout item to use the new webengine based rendering approach. 

## Risks

Low, but not zero. The previous research undertaken has determined that the approach is sound, but there's likely
still remaining shortcomings in the webengine classes which have not yet been identified.

## Performance Implications

N/A
## Backwards Compatibility

N/A
