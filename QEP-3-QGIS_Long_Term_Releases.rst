.. _qep3:


QGIS Enhancement 3: Long term releases
======================================

:Date: 2014/10/03
:Author: Tim Sutton
:Contact: tim@kartoza.com
:Author: -
:Contact: -
:Last Edited: 2014/10/03
:Status:  Final Draft
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

General procedure
.................

It is proposed to adopt the following workflow for the release cycle:

* We will add a new branch 'stable' which will contain the current stable release.
* The existing cadence of 3 releases per annum will be continued
* Each 3rd release will be designated a long term release (LTR), which will be supported for the year that follows.
* The feature / freeze window for the LTR will be an inversion of the normal release, with a 1 month feature addition window and a 3 month feature freeze for documenting the new release and stabilising it.
* On a monthly basis a bug fix release will be made. This will be conditional on there being one or more commits to the release branch within that month period.
* Developers will be invited to backport bug fixes to the stable branch
* Documentation team will only release the documentation once per annum - for the LTR - and spend their time during the year documenting new features as they arrive in git, and then the three month freeze window polishing the documentation to be ready with the release.



Workflow implications
.....................

.. image:: https://cloud.githubusercontent.com/assets/178003/4607065/ae1c0956-523e-11e4-95d9-cea1a3429faf.png

The diagram above illustrates the proposed workflow. The major impact will be on packaging (which is described below) 
but there will also be implications on the general git workflow. In particular we will need to maintain more branches
and developers will need to be conscious of which branch fixes etc. need to be ported to.

At any given time there will be three active branches:
 
* *the master branch* - this is where all new code should arrive into the code base.
* *the release branch* - this will be branched from the master branch every four months. The non LTR
  releases will be for giving users early access to new features in a known version of QGIS. Bug fixes will be 
  made against the release branch for the duration of their validity. The branch shall be named 
  by the major and minor components of the release number e.g. **2.6**.
* *the long term release (LTR) branch* - this will be branched from the community release branch after the first patch level
  release or 1 month, whichever is the soonest. The branch shall be named based on the branch from which
  it is derived with an additional 'LTR' suffix e.g. **2.6LTR**. The branch shall exist for 1 year, wherapon the next
  Long Term Release shall be made.


Packaging implications
......................

Unfortunately this rather simple proposal has rather large implications for packaging. 5 types of 
packages will be made available as part of our official QGIS packaging effort:

1) **Developer nightly packages.** These packages will be built on a nightly basis against the **master** branch.
   They are intended to provide early adopters and testers with easy access to an installer that they can use
   to test upcoming features and new fixes to issues in the development branch.
2) **Latest Release nightly packages.** These packages will be built on a nightly basis against the latest release
   branch. They are intended to provide testers and folks doing QA with a platform to test the upcoming even numbered 
   patch level release against the latest release branch.
3) **Latest Release packages.** These are release packages made at a fixed 4 month interval. They represent a snapshot
   of the development version with new features and an incremental feature release. The packages are released after a
   freeze period and in terms of stability reflect the second most stable packages we offer (after LTR packages).
4) **LTR nightly packages.** These are preview packages for the impending LTR patch level release. They are intended
   to provide a test environment for users and testers who wish to verify fixes that have made their way into the LTR
   branch work properly and have not introduced any side effects.
5) **LTR packages.** These are our 'best work' packages which provide a patch level package on the current Long Term
   Release.
   
For end-users we will simplify the choice by suggesting they download Long Term Release or the Latest packages.



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

This QEP serves as the documentation for the LTR procedure, and will be migrated
to the project governance documentation.

Issue Tracking ID(s)
--------------------

(required)




Voting History
--------------

(required)
