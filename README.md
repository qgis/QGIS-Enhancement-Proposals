QEP Process and workflow
---

QEPs (QGIS Enhancement Proposals) are used in the process of creating and discussing new enhancements or policy for
QGIS

QEPs should generally be created for (only examples):

- Large application wide changes (e.g major UI redesign)
- Large work with wider scope (e.g something that might effect how layers are loaded)
- Community processes and policies (e.g 3.0 release, changes to coding conventions)

Generally smaller features do not require a QEP unless they can have large knock on effect.

QEPs which describe project policies (such as those which describe coding conventions, acceptable practice for Pull
Request reviews, etc) are mutable, and can be revised in future. See Processes and Policies section below for a
description on how to amend an existing QEP.

## Process and policies

If you are:

- **Proposing a brand new QEP**: make a copy of the [QEP-0-Template.md](QEP-0-Template.md) file, populate the template
  and rename as required. Submit a Pull Request to this repository adding your new QEP file.
- **Proposing a change to an existing QEP**: just submit a Pull Request which modifies the corresponding QEP markdown
  file accordingly.

The same policies apply whether you a proposing a new QEP or a change to an existing QEP:

- The proposal must be open for at least two weeks. This may be extended upon request (eg, "I'm on holidays but have
  feedback to give").
- Proposals (both for new QEPs and amendments to existing QEPs) should be announced to the community via a message to
  the [QGIS Developer Mailing List](https://www.qgis.org/community/organisation/mailinglists/#qgis-developers-list).
  The [QGIS User Mailing List](https://www.qgis.org/community/organisation/mailinglists/#qgis-users-list) should also be
  messaged if the changes impact on QGIS end-users.
- Comments from the **whole** community are invited and valuable! You do not need any special permissions or status in
  order to give feedback. We welcome feedback from all parties, whether you're a developer, a documenter, a translator
  or an end-user.
- For acceptance, a proposal requires at least 2 +1's from core QGIS developers or PSC members
- If -1 votes are received from core QGIS developers, then the proposal should be amended or further discussion
  conducted to satisfy all interested parties. If consensus cannot be reached, the proposal can be raised to the PSC for
  voting.
- A PR can also be opened at the same time - *this is done at the developer's own risk as the proposal may be rejected
  and the development effort wasted*

When the discussion and voting period ends, one of the following actions are taken:

- **If the proposal was accepted**: the Pull Request should be amended, making sure that the QEP markdown file correctly
  reflects the final accepted form of the proposal. It should then be merged. Please announce
  to [QGIS Developer Mailing List](https://www.qgis.org/community/organisation/mailinglists/#qgis-developers-list) the
  outcome of the proposal.
- **If the proposal was rejected**: the Pull Request should be closed

## List of approved QEPs

### Policy/process related QEPs

- [#314 Code style and practice guidelines](https://github.com/qgis/QGIS-Enhancement-Proposals/blob/master/qep-314-coding-style.md). This
  QEP documents coding standards and conventions used throughout the QGIS code base. Developers are required to follow these standards
  when submitting code to QGIS.
- [#323 Change submission and merge policies](https://github.com/qgis/QGIS-Enhancement-Proposals/blob/master/qep-323-code-submission-policy.md). This QEP documents policies for submission of changes to the main QGIS code repository.
  Developers are required to follow these standards when submitting code to QGIS.
- [#320 Backport policies](https://github.com/qgis/QGIS-Enhancement-Proposals/blob/master/qep-320-backport-policy.md). This
  QEP documents the standards and requirements for backporting changes from the ``master`` branch to Long Term Release (LTR)
  and current stable release branches in QGIS. Developers must follow these guidelines when submitting backports.
- [#408 AI tool use policy](https://github.com/qgis/QGIS-Enhancement-Proposals/blob/master/qep-408-ai-tool-policy.md). This QEP documents the QGIS project policy regarding the use of AI/LLM tools by contributors.

## FAQ

### Do I need a QEP for a bug fix?

Nope. Just open a PR
