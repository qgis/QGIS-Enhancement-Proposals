.. _qep#[.#]:

============================
QGIS Enhancement #: QGIS 3.0
============================

:Date: 2015/10/21
:Author: Nyall Dawson
:Contact: nyall dot dawson at gmail dot com
:Last Edited: 2015/10/21
:Status:  Draft
:Version: QGIS 3.0

1. Summary
----------

QGIS 2.16 should become QGIS 3.0, and include:

- an API break
- Qt5 only build support
- Python 3.0 support
- An extended development cycle to facilitate these changes

2. Background
-------------


3. Proposed Changes
-------------------

3.1 API Break
.............

QGIS API has remained stable throughout the 2.0 series. While it is acknowledged
that this is desirable for plugin and script maintainers, QGIS 3.0 will allow 
API breaks. This is required to clean up the codebase and make deeper changes
and bug fixes which are currently impossible to do without breaking API. It
also allows for removal of old compatibility code (currently QGIS master has over
300 methods marked as deprecated), simplifying future development and 
maintenance efforts.

3.2 Qt 5 Requirement
....................

Currently QGIS supports building on both Qt4 and Qt5 libraries, with the default
build being Qt4. Qt4 has been end-of-lifed by it's developers and is no longer
maintained. This raises signficant risks for QGIS if we do not move to Qt5 builds
as quickly as possible. For instance, Qt4 cannot be built on the latest OSX 
release (10.11). Third party patches have been made which partly resolve this
issue, but the side effects are unknown. It is highly likely that a future
OSX release, Windows update or linux packaging change will prevent use of
Qt4 on one of our supported platforms and result in QGIS being unavailable
for that platform.

Additionally, numerous bugs present in the current QGIS release which are
caused by issues in Qt have been resolved in Qt5. Moving to Qt5 is the only way
to fix these issues.

There's also significant performance and feature benefits made possible by
the switch to the newer platform.

3.2.1 Issues with moving to Qt5
...............................

Qt5 has dropped support for the QWebView/QWebPage classes. The replacement
class, QtWebEngine, is immature and lacks several important features which
are currently used by QGIS. The largest impact this will have will be on
the composer "HTML" item. This item is currently rendered using a QWebPage
painting directly on to a QPainter surface, resulting in text rendering
as text objects and vectors. This is NOT currently possible with the 
replacement classes, and it is unclear whether this will be brought back
by a future Qt release. It is also impossible to render QtWebEngine contents
using a transparent background. Unfortunately, there is no workaround for these
limitations and the feature will have to be modified so that HTML content is either:

- rendered as an upscaled (read: pixelated) image
- rendered using the subset of HTML and CSS supported by QTextDocument (see
  http://doc.qt.io/qt-5/richtext-html-subset.html )

3.3 Python 3.0
..............

** I'm unclear about the exact requirement here - but I believe PyQt5 is
python 3.0 only, so our plugins would need to be ported to Python 3.0
also....

4.0 Timeline for 3.0
--------------------

In order to maximise the value of an API break, and to allow sufficient
time for the changes required to port to Qt5/Python 3.0, the 3.0 development
cycle should span twice the normal development cycle. This means that
the revised timeline for 2016 would be:

2.14 (LTR) 26.02.2016
2.16 - NO RELEASE
3.00 21.10.2016

#.# Affected Files
..................

Almost everything!

#. Further Considerations/Improvements
--------------------------------------

Support for plugin developers in the form of guides for porting to QGIS 3.0
will need to be developed.


#. Voting History
-----------------

(required)
