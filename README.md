QEP Process and workflow
---

QEP's (QGIS Enhancement Proposals) are used in the process of creating and discussing new enhancements or policy for QGIS

All QEP's are created as new [issues](https://github.com/qgis/QGIS-Enhancement-Proposals/issues).  Create a new ticket using the [template](https://raw.githubusercontent.com/qgis/QGIS-Enhancement-Proposals/master/QEP-0-Template.md)

QEP's should generally be created for (only examples):

- Large application wide changes (e.g major UI redesign)
- Large work with wider scope (e.g something that might effect how layers are loaded.)
- Community processes and policies (e.g 3.0 release)

Generally smaller features do not require a QEP unless they can have large knock on effect.

QEP numbers will be assigned by [@NathanW2](https://github.com/NathanW2) when issues are created. Ping him if one is not assigned.  


## Process

- Must be open for at least one week
- PR can also be opened at the same time - however **not** recommend if something is still in planning stage and changing, or chance of rejection e.g don't update all the code headers with MIT and then open a QEP because it wouldn't happen.
- Code based QEPs require at least 2 +1's from core developers. Including +1 from maintainer of that area of code (e.g Nyall for composer work, Martin for rendering thread)
- Project QEPs require majority PSC vote
- May be extended upon request (eg, I'm on holidays but have feedback to give)
- May be extended if required 
- Others can assign themselves as interested e.g I might be interested in Processing but can't comment on the code. Mainly just to provide feedback.
- If no maintainer for a area of code, requires at least 2 +1s from core developers.
- If -1 votes are received (for code based QEPs) then the proposal should be amended or further discussion conducted to satisfy all interested parties. If consensus cannot be reached, the QEP can be raised to the PSC for voting.


## FAQ

### Do I need a QEP for a bug fix?

Nope. Just open a PR or push directly if you have commit rights
