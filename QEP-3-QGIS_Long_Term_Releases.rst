.. _qep3:


QGIS Enhancement 3: Long term releases
======================================

:Date: 2014/10/03
:Author: Tim Sutton
:Contact: tim@kartoza.com
:Author: -
:Contact: -
:Last Edited: 2014/10/03
:Status:  Draft
:Supercedes: -
:Version: QGIS 2.X
:Sponsor:
:Sponsor URL:

.. note::

    See :ref:`QEP 1 <qep1>` for description of QEP process.

Summary
-------

This proposal is aimed at providing more stable (as in seldom changing) releases
of QGIS for corporate and institutional users who prefer a slower release
cycle with a more stable version of QGIS that receives bug fix updates
over the life of the release.


Proposed Change
---------------

It is proposed to adopt the following workflow for the release cycle:

* We will add a new branch 'stable' which will contain the current stable release.
* The existing cadence of 3 releases per annum will be continued
* Each 3rd release will be designated a long term release (LTR), which will be supported for the year that follows.
* The feature / freeze window for the LTR will be an inversion of the normal release, with a 1 month feature addition window and a 3 month feature freeze for documenting the new release and stabilising it.
* On a monthly basis a bug fix release will be made. This will be conditional on there being one or more commits to the release branch within that month period.
* Developers will be invited to backport bug fixes to the stable branch
* Documentation team will only release the documentation once per annum - for the LTR - and spend their time during the year documenting new features as they arrive in git, and then the three month freeze window polishing the documentation to be ready with the release.


Further Considerations
----------------------

Life span of LTR's
..................

The intent of this QEP is to provide an alternative for people who prefer
a slower release cycle. Currently I am only proposing to support each LTR
release for one year and that there should be no concurrent LTR releases. As
such we would only ever need to maintain one LTR release.

Policy for patches
..................

The criteria for creating / backporting fixes to the stable branch should be as
follows:

* The patch should introduce no regressions to the test suite.
* The patch should not alter the API except in cases where the API is
  broken and the patch fixes it.
* The patch should not change the user interface except in cases where
  there is an error in the user interface and the patch fixes it.
  
.. note:: No new features will be allowed in the LRT after it is released.


Documentation
-------------

This QEP serves as the documentation for the LRT procedure, and will be migrated
to the project governance documentation.

Issue Tracking ID(s)
--------------------

(required)




Voting History
--------------

(required)
