.. _qep#[.#]:

==========================================================
QGIS Enhancement #: Compulsory Unit Tests for Core Changes
==========================================================

:Date: 2014/10/05
:Author: Nyall Dawson
:Contact: nyall.dawson@gmail.com
:Last Edited: 2014/10/05
:Status:  Draft
:Version: QGIS 2.7+

.. note::

    See :ref:`QEP 1 <qep1>` for description of QEP process.

#. Summary
----------

This proposal is aimed at providing more stable releases and preventing regressions in QGIS releases by expanding
the coverage of unit tests and adopting a hard line approach to compulsory unit tests.

#. Background
-------------

The QGIS coding guidelines (ref) state that since "November 2007 we require all new features going into master to be
accompanied with a unit test" (https://github.com/qgis/QGIS/blob/master/CODING#L738). However, this requirement is not
being enforced and the vast majority of core changes are not accompanied by unit tests. Lack of unit tests impact
on our ability to deliver stable, regression free releases and potentially harms our reputation as an enterprise-ready GIS.

#. Proposed Solution
--------------------

From QGIS >=2.7, all commits which modify code in "core" are required to be accompanied by sufficient unit tests to ensure
that they will not suffer regressions in future QGIS releases. Commits which are not accompanied by unit tests will not be accepted
into master. Code changes for GUI and app are immune from this requirement due to the complexity in writing interactive
tests suites. 

Exemption for specific commits may be granted, but only after discussion on the QGIS developer list.

Generally, unit tests in either c++ or python are acceptable (or both). However, methods which involve transfer of ownership
of objects must include python tests to ensure that this transfer is handled correctly.

To enforece this requirement, commits which modify core and which are not accompanied by unit tests (or have been granted
exemption via the Dev list) will be reverted. Pull requests without sufficient unit tests will not be considered for merging.


#. Implementation Details
-------------------------

If accepted, the existing requirement for unit tests in the CODING file will be updated to reflect the new requirements. The
new requirements will be advertised to existing developers via an announcement on the QGIS-Dev mailing list.

#. Voting History
-----------------

(required)
