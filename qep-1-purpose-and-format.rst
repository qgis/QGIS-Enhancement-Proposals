Draft - QEP (QGIS Enhancement Proposal) 
================================

- Type: Process
- Author: Nathan Woodrow

Wait? Is this QEP a proposal to use QEPs?
------------------------------------------

Indeed it is. This is QEP 1 which is itself a QEP. 

Problem
------------------------------------------

QGIS is a fast growing project and such it can be hard to control the flow of new features and how they interact.  New features can sometimes conflict with other semi complete or complete features which can cause issues when using QGIS.   As a project QGIS needs a formal process to propose and vote for major changes which can require more design time and structure then a quick fix.

This proposal is for a QEP (QGIS Enhancement Proposal) to formalize the process. QEPs will also serve as a reference for why something was, or wasn't, accepted.and why.

What is QEP?
------------------------------------------
A QEP (QGIS Enhancement Proposal) is a proposal and design document for change to a QGIS or the way the project is operating as a whole.  It is basically a RFC (Request for Change) but with wider scope and more focus on QGIS.  A QEP can be a design for code and UI changes, or it can be more general and project based for something changing the bug tracker or website design for example.

A QEP is designed to be a single location for overview of a requested change and to make sure the change will fit into the future of the project. Creating and submitting a QEP does not guarantee acceptance.  QEPs are designed to filter ideas and refine them before they enter the project, particularly if they could have long term effects or breaking changes, e.g breaking API.   

QEP Types?
------------- 
There are three kinds of a QEPs
 - **Technical (API)** - A Technical code and API focused proposal.  These are normally related to the core project itself and more focus around the code structure . 
 - **GUI / Workflow (UX)** - A UI, or application workflow focused proposal. These are related to how QGIS looks and preforms to the user.  These may not require new code but might be full UI reworks.
 - **Process** - A QEP that is a proposal to change a process which the community around QGIS follow s. This could be something like changing the bug tracker, changing website platform, etc.

Who can discuss a QEP?
-----------------------

Discussion around a QEP is open to eveyone regardless of position within the project.  Discussion can be undertaken within the QEP proposal on GitHub.

Who can vote?
-------------

Only a set allocated pool of people can formally vote on a QEP for it to be accepted.  The pool depends on the type of QEP.

- **Technical (API)** - A Technical QEP can be voted on by the following members: xxxxxxx
- **GUI / Workflow (UX)** - A GUI / Workflow QEP can be voted on by the following members: xxxxxxx
- **Process** -  A process QEP is voted by the PSC exclusively and only the PSC.

Submitting a QEP?
------------------

A QEP should follow a set of guidelines when being written.  This is to ensure all are written in the same format and are easy to follow. 

As a general rule one should consult the mailing list to consider community opinion before opening a QEP.  This will reduce the chance of a QEP being rejected.
