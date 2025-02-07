# QGIS Enhancement 320: Backport policies

## Summary

This QEP documents the standards and requirements for backporting changes from the ``master`` branch to
Long Term Release (LTR) and current stable release branches in QGIS. Developers must follow these guidelines
when submitting backports.

Notes:

- These guidelines describe QGIS specific backporting practices only. Following general
  "best practice" for software maintenance and version control is assumed.
- Different policies apply for backports to stable release branches versus LTR branches.

## General Backporting Guidelines
 
- 1.1. Changes must be merged to ``master`` before backporting.
- 1.2. Backports must be submitted using Pull Requests.
- 1.3. A backport must be approved by a core QGIS developer prior to merge.
  - 1.3.1. The developer of the original change is not allowed to "self-approve"
   their own backported changes.
- 1.4. Changes which are permissible for backporting are:
  - 1.4.1. Bug fixes
  - 1.4.2. API modifications and additions which have a low risk of introducing regressions
  - 1.4.3. Test case additions
  - 1.4.4. Documentation fixes and improvements
  - 1.4.5. Performance optimizations
- 1.5. Changes which are NOT permissible for backporting include:
  - 1.5.1 New features
- 1.6. Try to avoid backporting changes which introduce new strings for translation,
  or modifications to existing string translations. These usually result in an English
  language string appearing within QGIS, which is undesirable.
- 1.7. Exercise common sense to decide whether a large and intrusive change is
  suitable for backporting.

### 2. Stable Release (non-LTR) Branch Backports

- 2.1. All changes which satisfy the General Backporting Guidelines are permissible for
  immediate backporting to the stable release branch.

### 3. LTR Branch Backports

- 3.1. Backports to the LTR branch must first target the `queued_ltr_backports` branch.
  - 3.1.1. Changes in `queued_ltr_backports` are merged to the current LTR release branch
    after one point release cycle delay. This ensures that the changes have been included
    in a stable release for at least one cycle prior to inclusion in the LTR, significantly
    reducing risk of mid-LTR regressions.
    - 3.1.1.1. In the case that a regression is identified in a queued LTR backport, either
      the backport should be reverted from the LTR branch OR case taken to ensure a regression
      fix is merged for the same LTR release the original buggy backport.
  - 3.1.2. Exceptions are made for the following backports, which are permissible for 
    immediate backporting into the LTR branch:
    - 3.1.2.1. Crash fixes
    - 3.1.2.2. Data corruption fixes
    - 3.1.2.3. Regression fixes
    - 3.1.2.4. Security related fixes
