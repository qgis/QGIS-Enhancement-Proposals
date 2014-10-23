.. _qep#[.#]:

========================================================================
QGIS Enhancement 1: QGIS QEP process definition
========================================================================

:Date: 2014/08/26
:Author: Nathan Woodrow
:Contact: woodrow dot nathan at gmail dot com
:Author: Pirmin Kalberer
:Status:  Draft
:Version: QGIS All

Summary
------------------------------------------

This proposal is for a QGIS QEP to formalize the process for a change to the QGIS project.


Rationale
------------------------------------------

QGIS is a fast growing project and such it can be hard to control the flow of new features and how they interact. New features can sometimes conflict with other semi complete or complete features which can cause issues when using QGIS. As a project QGIS needs a formal process to propose and vote for major changes which can require more design time and structure then a quick fix.
QEPs will also serve as a reference for why something was or wasn't accepted and why.


Implementation details
------------------------------------------

An QEP is a proposal and design document for a change to QGIS or the way the project is operating as a whole. An QEP can be a design for code and UI changes, or it can be more general and project based for something changing the bug tracker or website design for example.

An QEP is designed to be a single location for overview of a requested change and to make sure the change will fit into the future of the project. Creating and submitting an QEP does not guarantee acceptance. QEPs are designed to filter ideas and refine them before they enter the project, particularly if they could have long term effects or breaking changes, e.g breaking API.

As a general rule one should consult the mailing list to consider community opinion before opening an QEP. This will reduce the chance of an QEP being rejected.

Process
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In brief the project team votes on proposals submitted as pull requests on this git repository.

Detailed Process
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Proposals are written up and submitted as pull requests on this git repository and announced on the qgis-developer mailing list.
- Proposals are open for discussion and voting by any interested party, not just committee members, however votes by non PSC members only count as support for the idea not in the final score.
- Anyone may comment on proposals on the pull request, but only members of the Project Steering Committee's votes will be counted.
- While in a draft stage a QEP can be open with no time limit as the idea evolves.
- Complex QEPs are recommended to stay in draft for longer then simple ones in order for feedback to collected.
- QEP's which jump to final draft and voting might risk rejecting if to complex or not detailed enough


Voting
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- The QEP manager will make the offical call for voting on PSC list. 
- Voting can only take place once the QEP is in final draft mode 
- Voting is only open for seven days

Only the PSC can take a final vote on the proposal before it is accepted.  The voting weights are as follows:

	- "+1" to indicate support for the proposal and a willingness to support implementation.
	- "-1" to veto a proposal, but must provide clear reasoning and alternate approaches to resolving the problem within the seven days.
	- -0 indicates mild disagreement, but has no effect. 
	- 0 indicates no opinion
	- +0 indicate mild support, but has no effect.

- A proposal will be accepted if it receives +3 and no vetos (-1).
- Upon completion of discussion and voting the proposer should announce whether they are proceeding (proposal accepted) or are withdrawing their proposal (vetoed).
- The Chair adjudicates in cases of disputes about voting.

QEP Manager
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Current Manager: Nathan Woodrow

The QEP manager controls the flow of QEPs that are in the work and the overall system.

The QEP manager has the following rights:

- Make the offical call for voting on PSC list. 
- Move a QEP from `Draft` to `Final Draft` (normally with support from community) [1]
- Move a QEP from `Final Draft` to `Draft` if not enough information is provided [1]

[1] The community and PSC can also call on the manager to promote/demote a QEP 

Miscellaneous
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The document is an adapted version of the GDAL/MapServer RFC process.

.. note::

    See :ref:`QEP 0` for template QEP document.


Voting history
------------------------------------------

