# QGIS Enhancement: Plugins Security and QA validator on plugins.qgis.org

**Date** 2026/01/20

**Author** Lova Andriarimalala (@Xpirix)

**Contact** lova at kartoza dot com

**Version** QGIS Plugins Website

# Summary

The QGIS Plugins Repository continues to grow and has recently surpassed 3,000 plugins, including those currently under review.

With this growth, it is increasingly important to strengthen our security measures and ensure that malicious or low-quality plugins are not distributed through QGIS infrastructure.

To address this, new Security and Quality Assurance (QA) checks have been introduced in the QGIS Plugins Website (see https://github.com/qgis/QGIS-Plugins-Website/pull/219). These checks are automatically executed whenever a new plugin or a new plugin version is uploaded.

This QEP proposes to make these checks mandatory and blocking: any new plugin or plugin version that triggers at least one critical security issue would be rejected and not published in the repository until the issue is resolved.

## New security and QA checks

There are currently five checks that will run when uploading a new plugin version:
- Bandit Security Analysis: Professional security vulnerability scanner for Python code (checks for SQL injection, hardcoded passwords, unsafe functions, etc.)
- Secrets Detection: Scans for hardcoded secrets, API keys, passwords, and tokens using detect-secrets
- Code Quality (Flake8): Python code quality and style checker.
- File Permissions: Checks for files with executable or unusual permissions
- Suspicious Files: Detects suspicious file types, hidden files, or unexpected executables

The result is detailed under the Plugin Version Details Page > Security Scan tab.
More details about these are available at https://plugins.qgis.org/docs/security-scanning

The proposition for now is to make the **Bandit Security Analysis** and **Secrets Detection** a blocking validator if at least one critical issue is found.


## Bandit Security Analysis

When a critical vulnerability issue or malicious code is found, this check will block a new plugin/version from being uploaded and show more details about it:

![](./images/qep408/bandit-check.png)


## Secrets Detection

When a hardcoded secret, API key... is found, this check will block a new plugin/version from being uploaded and show more details about it:

![](./images/qep408/secrets-check.png)

## Other checks
Code quality, file permissions and suspicious files checks will stay as non-blocking validators and be available on the Plugin Version Details Page > Security Scan tab.

![](./images/qep408/other-checks.png)

## Risks

Some existing plugins will encounter the blocking validator when uploading a new version. However, we have implemented these checks as a soft validator, so they can be fixed and avoid being blocked later.

## Performance Implications

None

## Further Considerations/Improvements

None

## Issue Tracking ID(s)

https://github.com/qgis/QGIS-Plugins-Website/issues/213
