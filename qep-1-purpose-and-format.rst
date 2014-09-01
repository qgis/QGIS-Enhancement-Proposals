QGIS RFC 1: QGIS RFC process definition
================================

- Date: 2014/08/26
- Author: Nathan Woodrow / Pirmin Kalberer
- Status: Draft
- QGIS Version: -

Summary
------------------------------------------

This proposal is for a QGIS RFC to formalize the process for a change to the QGIS project.


Rationale
------------------------------------------

QGIS is a fast growing project and such it can be hard to control the flow of new features and how they interact. New features can sometimes conflict with other semi complete or complete features which can cause issues when using QGIS. As a project QGIS needs a formal process to propose and vote for major changes which can require more design time and structure then a quick fix.
RFCs will also serve as a reference for why something was or wasn't accepted and why.


Implementation details
------------------------------------------

An RFC is a proposal and design document for a change to QGIS or the way the project is operating as a whole. An RFC can be a design for code and UI changes, or it can be more general and project based for something changing the bug tracker or website design for example.

An RFC is designed to be a single location for overview of a requested change and to make sure the change will fit into the future of the project. Creating and submitting an RFC does not guarantee acceptance. RFCs are designed to filter ideas and refine them before they enter the project, particularly if they could have long term effects or breaking changes, e.g breaking API.

As a general rule one should consult the mailing list to consider community opinion before opening an RFC. This will reduce the chance of an RFC being rejected.

Process
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In brief the project team votes on proposals submitted as pull requests on this git repository. Proposals are available for review for at least two days, and a single veto is sufficient to delay progress though ultimately a majority of members can pass a proposal.

Detailed Process
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Proposals are written up and submitted as pull requests on this git repository and announced on the qgis-developer mailing list.
- Proposals are open for discussion and voting by any interested party, not just committee members
- Proposals need to be available for review for at least two business days before a final decision can be made.
- Respondents may vote "+1" to indicate support for the proposal and a willingness to support implementation.
- Respondents may vote "-1" to veto a proposal, but must provide clear reasoning and alternate approaches to resolving the problem within the two days.
- A vote of -0 indicates mild disagreement, but has no effect. A 0 indicates no opinion. A +0 indicate mild support, but has no effect.
- Anyone may comment on proposals on the pull request, but only members of the Project Steering Committee's votes will be counted.
- A proposal will be accepted if it receives +2 (including the proposer) and no vetos (-1).
- If a proposal is vetoed, and it cannot be revised to satisfy all parties, then it can be resubmitted for an override vote in which a majority of all eligible voters indicating +1 is sufficient to pass it. Note that this is a majority of all committee members, not just those who actively vote.
- Upon completion of discussion and voting the proposer should announce whether they are proceeding (proposal accepted) or are withdrawing their proposal (vetoed).
- The Chair adjudicates in cases of disputes about voting. 

Miscellaneous
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The document is an adapted version of the GDAL/MapServer RFC process.

This RFC also serves as a template for future RFC's.


Compatibility
------------------------------------------

N/A


Voting history
------------------------------------------

+1 from ...
