QEP Process and workflow
---

QEP's (QGIS Enhancement Proposals) are used in the process of creating and discussing new enhancements or policy for QGIS

All QEP's are created as new [issues](https://github.com/qgis/QGIS-Enhancement-Proposals/issues).  Create a new ticket using the [template](https://raw.githubusercontent.com/qgis/QGIS-Enhancement-Proposals/master/QEP-0-Template.md)

QEP's should generally be created for (only examples):

- Large application wide changes (e.g major UI redesign)
- Large work with wider scope (e.g something that might effect how layers are loaded.)
- Community processes and policies (e.g 3.0 release)

Generally smaller features do not require a QEP unless they can have large knock on effect.

## Process

- A QEP must be open for at least two weeks. This may be extended upon request (eg, "I'm on holidays but have feedback to give").
- Newly submitted QEPs should be announced to the community via a message to the [QGIS Developer Mailing List](https://www.qgis.org/community/organisation/mailinglists/#qgis-developers-list). The [QGIS User Mailing List](https://www.qgis.org/community/organisation/mailinglists/#qgis-users-list) should also be messaged if the changes impact on QGIS end-users.
- Comments from the **whole** community are invited and valuable! You do not need any special permissions or status in order to give feedback. We welcome feedback from all parties, whether you're a developer, a documenter, a translator or an end-user.
- For acceptance, a QEP requires at least 2 +1's from core QGIS developers or PSC members
- If -1 votes are received from core QGIS developers, then the proposal should be amended or further discussion conducted to satisfy all interested parties. If consensus cannot be reached, the QEP can be raised to the PSC for voting.
- A PR can also be opened at the same time - *this is done at the developer's own risk as the proposal may be rejected and the development effort wasted*

## FAQ

### Do I need a QEP for a bug fix?

Nope. Just open a PR
