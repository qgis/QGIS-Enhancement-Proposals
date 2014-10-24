.. _qep#[.#]:

========================================================================
QGIS Enhancement 1: QGIS QEP process definition
========================================================================

:Date: 2014/10/13
:Author: Nathan Woodrow
:Contact: woodrow dot nathan at gmail dot com
:Author: Pirmin Kalberer
:Status:  Final Draft
:Version: QGIS All

Summary
------------------------------------------

This proposal is for a QGIS QEP to formalize the process for a change to the QGIS project.


Rationale
------------------------------------------

QGIS is a fast growing project and as such it can be hard to control the flow of new features and how they interact. New features can sometimes conflict with other semi complete or complete features which can cause issues when using QGIS. As a project QGIS needs a formal process to propose and vote for major changes which can require more design time and structure then a quick fix.
QEPs will also serve as a reference for why something was or wasn't accepted and why.


Implementation details
------------------------------------------

A QEP is a proposal and design document for a change to QGIS or the way the project is operating as a whole. A QEP can be a design for code and UI changes, or it can be more general and project based for something changing the bug tracker or website design for example.

A QEP is designed to be a single location for overview of a requested change and to make sure the change will fit into the future of the project. Creating and submitting a QEP does not guarantee acceptance. QEPs are designed to filter ideas and refine them before they enter the project, particularly if they could have long term effects or breaking changes, e.g breaking API.

As a general rule one should consult the mailing list to consider community opinion before opening a QEP. This will reduce the chance of an QEP being rejected.

Process
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In brief the PSC votes on proposals submitted as pull requests on this git repository.

Once a pull request is open it will be assigned a number by the QEP manager. This number will be reflected in the QEP document, pull request header, and wiki list of past and current QEPs.

Detailed Process
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Proposals are written up and submitted as pull requests on this git repository and announced on the qgis-developer mailing list.
- Proposals are assigned a QEP number by the manager.
- Proposals are open for discussion and feedback by any interested party, not just committee members (only votes by PSC members count for approval or denial of the proposal in the voting round)
- While in `Draft` a QEP can be open with no time limit as the idea evolves.
- Once in `Final Draft` no major changes should take place in the QEP.  Only final draft QEPs can be voted on.
- Complex QEPs are recommended to stay in draft for longer than simple ones in order for community feedback and support to collected.
- A QEP can **only** move from `Draft` to `Final Draft` via the QEP Manager

Final Draft
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A QEP in final draft should not undergo any major changes as it is considered stable at this point. 

Depending on the scale of the QEP it may sit in Final Draft mode for at least seven (7) days before being called for voting.

If no comments are received which affect the proposal during this seven day period the QEP will be called for voting.

Asking for more time to review and comment on a QEP, even if in draft and final draft mode, is acceptable. However it will be up to the QEP manager to decide if  something is dragging on with no meaningful feedback.

Voting
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- The QEP manager will make the official call for voting on PSC list. 
- Voting can only take place once the QEP is in final draft mode 
- Voting is only open for seven days

Only the PSC can take a final vote on the proposal.  The voting weights are as follows:

   - +1 to indicate support for the proposal.
   - 0 indicates no opinion
   - -1 to veto a proposal, but must provide clear reasoning.

After voting the rules are as follows:

- **Rejected** and back into `Draft` if more than two -1 votes are cast
- **Accepted** if majority of non zero votes are +1 and no more than two -1 votes have been cast.
- **Draft** if all 0 votes are cast - all 0 votes in this case indicate no strong support, but no strong veto arguments.  This might indicate the QEP requires more changes for PSC acceptance.   

- A `Draft` QEP with more then -2 can be fully rejected at the discretion of the PSC e.g QEP to change licence would be fully rejected as it will not happen.  

*Note:* Feedback from the PSC should be taken on board when back in draft mode. 

Upon completion of discussion and voting the proposer should announce whether they are:

- proceeding (proposal accepted) 
- fully withdrawing their proposal (vetoed).
- reviewing their proposal (vetoed or all 0 votes).

QEP Manager
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Current Manager: Nathan Woodrow

The QEP manager controls the flow of QEPs that are in the queue and the overall QEP process.

The QEP manager has the following rights:

- Make the offical call for voting on PSC list on QEP.
- Move a QEP from `Draft` to `Final Draft` 
- Move a QEP from `Final Draft` to `Draft`

All actions are normally done when there is support from the community to do so.

The community and PSC can also call on the manager to promote/demote a QEP. 

Miscellaneous
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The document is an partly adapted from of the GDAL/MapServer RFC process.

.. note::

    See :ref:`QEP 0` for template QEP document.


Voting history
------------------------------------------

