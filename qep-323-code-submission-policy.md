# QGIS Enhancement 323: Change submission and merge policies

## Summary

This QEP documents policies for submission of changes to the main QGIS code repository.
Developers are required to follow these standards when submitting code to QGIS.

## Policies

### 1. Change submission

- 1.1. All changes must be submitted in the form of Pull Requests via GitHub.
  - 1.1.1. Pull Requests can be made by **ANYONE**, and **EVERYONE** is encouraged to
    contribute to QGIS coding! No special permission or status is required to submit
    a change to QGIS.
  - 1.1.2. Direct push to master is permitted in **extreme circumstances** only. These are:
    - String fixes for code comments or other non-functional text. (This does not include
      code documentation, changes to the README files, or other documentation).
    - Fixes for the CI environment or GitHub workflows where the lengthy run-time of
      CI checks on pull requests is prohibitive, or where there is a need to repair
      CI environment in order to allow normal pull request activity.
    - Release and packaging related commits, such as version tagging.
  - 1.1.3. All Pull Requests should be accompanied by a descriptive title and explanatory text.
    Linking to existing issues, feature requests or external discussion are encouraged
    for reviewer context. 
- 1.2. All code submissions must be accompanied by appropriate unit tests.
  - 1.2.1. All changes to the ``core``, ``analysis``, or ``server`` modules must
    have extensive unit testing.
    - 1.2.1.1. Exceptions are made where the effort required in mocking interfaces is excessive,
      or where tests are likely to be flaky and trigger false-positives on the CI environment.
  - 1.2.2. Changes to the ``gui`` or ``app`` modules should have as many tests as are possible
    without extensive GUI interface mocking.
    - 1.2.2.1. There are no expectations for testing of keyboard or mouse interaction with
      dialogs and widgets.
  - 1.2.3. Changes to data providers must be accompanied by appropriate unit tests, unless
    the effort required to mock external interfaces is excessive.
- 1.3. Code changes must abide by the [coding style policies](qep-314-coding-style.md).

### 2. Pull Request Review and Merging

Before a Pull Request can be merged to the repository, it must:

- 2.1. Pass all existing CI checks.
- 2.2. Discussions and comments from any interested party are encouraged. No special
  status or permissions are required to participate in the review discussion.
- 2.3. Before merge, the Pull Request **MUST** be reviewed and approved by a **Core Contributor**
  with repository write permission.
  - 2.3.1. Developers are **NOT** permitted to self-approve Pull Requests they have submitted.
    - 2.3.1.1. An exception is made where a developer has submitted a "fixed up" version
      of someone else's Pull Request, e.g. where the original submitter has not responded
      to changes from the reviewer or where the original submitter has indicated that they
      are otherwise unable to make the required changes themselves.
- 2.4. Have no outstanding change requests or unresolved discussion from a **Core Contributor**.
