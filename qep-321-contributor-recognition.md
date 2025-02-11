# QGIS Enhancement 321: Contributor recognition and commit rights policies

The QGIS project splits the concept of "notable developer recognition" from those with commit rights to the code
repository. This is done in order to maintain good security practice for the code repository, with only those who
are currently actively contributing to QGIS code having push/merge rights to the repository.

Accordingly, there are two different types of QGIS developer status:

## 1. QGIS Core Contributors

"Core contributor" status is a lifetime status granted to developers who have demonstrated ongoing commitment and
excellence in QGIS development.

In order to be eligible, a developer must have:

- 1.0.1. A history of high-quality code contributions to QGIS for a period of at least 12 months
- 1.0.2. Demonstrated a high level of knowledge of c++, Python, the QGIS API, and QGIS development processes and practices
- 1.0.3. Demonstrated a willingness to work in a friendly, collaborative manner and respond positively to change requests
  and suggestions from other core contributors

(While the project recognises the **immense** value in non-code contributions to QGIS, this status is designed exclusively for
contributors of code.)

## 1.1. Process for Nomination

TODO

# 2. Commit Rights

This group of developers are those with write access to the main QGIS code repository. Unlike the "core contributor"
status, this is a transient right and reflects only a developer's **current** activity within the QGIS project.
**It is not a reflection of a developer's abilities, trustworthiness or skill, and is considered purely an
administrative status designed to minimize security risk to the project while permitting responsive project
maintenance.**

- 2.0.1. Only developers who have previously been awarded the "core contributor" status are eligible for commit rights.
- 2.0.2. When a new developer is initially awarded the "core contributor" status, they will also be granted commit rights
to the code repository. However, QGIS developers may find other demands on their time or life circumstances and
naturally reduce their involvement in the project. Over time, inactive accounts can represent a security risk,
and maintaining accurate contact information becomes challenging. For these reasons, the following commit rights policy
apply:

## 2.1. Commit Rights Expiration

- 2.1.1. Committers who have not made significant code contributions to the project **in the last 12 months** will have their
commit rights automatically removed.

When commit rights are removed:

- 2.1.2. The committer will be notified via their registered email address
- 2.1.3. Their **write** access to the code repository will be revoked

All developers are encouraged to continue to make contributions to QGIS via pull requests, regardless of whether
or not they currently hold (or have previously held) commit rights!

## 2.2. Reinstatement

- 2.2.1. Committers can request the reinstatement of their commit rights by contacting the QGIS PSC and

  - Demonstrating renewed active participation in the project
  - Demonstrating **up-to-date** knowledge of the QGIS codebase, API, and development practices
  - Demonstrating a strong need for commit rights

## 2.3. Notification Requirement

Active committers agree to:

- 2.3.1. Notify the QGIS PSC **immediately** of any change in circumstances affecting their ability to contribute regularly
- 2.3.2. Update their contact information when necessary
- 2.3.3. Acknowledge that their commit rights may be revoked as per this policy

## 2.4. Special Circumstances

The QGIS PSC reserves the right to:

- 2.4.1. Remove commit rights immediately in cases of security concerns or code of conduct violations
- 2.4.2. Grant exceptions to this policy in special cases, documented with clear justification
- 2.4.3. Review and update this policy as needed to maintain project security and vitality
- 2.4.4. In recognition of the project's history, Gary Sherman is considered an extraordinary exception to these policies and
  will always maintain write access to the code repository without expiration.
