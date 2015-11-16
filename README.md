QEP Process and workflow
---

QEP's (QGIS Enhancement Proposals) are used in the process of creating and discussing new enhancements for QGiS.

Currently Github is used to centralize the discussion and edit's to a proposal.

All QEP's area created as new [issues](https://github.com/qgis/QGIS-Enhancement-Proposals/issues).  Create a new ticket using the [template](https://raw.githubusercontent.com/qgis/QGIS-Enhancement-Proposals/master/QEP-0-Template.md)

QEP's should generally be created for (only examples):

- Large application wide changes (e.g major UI redesign)
- Large worker with wider scope (e.g something that might effect how layers are loaded.)
- Community process and policys (e.g 3.0 release)

Generally smaller features do no require a QEP unless they can have large knock on effect.

QEP numbers will be assigned by [@NathanW2](https://github.com/NathanW2) when issues is created. Ping him if one is not assigned.  


## Process

- Must be open for at least one week
- PR can also be opened at the same time - however **not** recommend if something is still in planning stage and changing, or chance of rejection e.g don't update all the code headers with MIT and then open a QEP because it wouldn't happen.
- Code based QEPs require at least 2 +1's
- Project QEPs require majority PSC vote
- Requires +1 from maintainer of that area of code e.g Nyall for composer work, Martin for rendering thread
- May be extended upon request (eg, I'm on holidays but have feedback to give)
- May be extended if required 
- Others can assign themselves as interested e.g I might be interested in Processing but can't comment on the code.  Mainly just to provide feedback.
- If no maintainer for a area of code, requires at least 2 +1s from core devs.
