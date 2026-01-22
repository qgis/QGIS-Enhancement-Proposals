# QGIS Enhancement: Plugins Security and QA validator on plugins.qgis.org

**Date** 2026/01/20

**Author** Lova Andriarimalala (@Xpirix)

**Contact** lova at kartoza dot com

**Version** QGIS Plugins Website

# Summary

The QGIS Plugins Repository continues to grow and has recently surpassed 3,000 plugins, including those currently under review.

With this growth, it is increasingly important to strengthen our security measures and ensure that malicious or low-quality plugins are not distributed through QGIS infrastructure.

To address this, new Security and Quality Assurance (QA) checks have been introduced in the QGIS Plugins Website (see https://github.com/qgis/QGIS-Plugins-Website/pull/219) as a soft validator.

This QEP proposes running all security and QA checks asynchronously after plugin upload as a blocking validator. Developers get instant upload confirmation, then receive email results when checks complete.

**Important:** Plugins won't be available for approval or download until:
1. All checks complete successfully, AND
2. No critical security issues found (Bandit or Secrets Detection)

For trusted users who normally get auto-approval, this still applies - auto-approval only happens if checks pass.

This approach gives fast uploads without compromising security, and approvers are automatically notified when plugins are ready for review.

## New security and QA checks

There are currently five checks that will run when uploading a new plugin version:
- Bandit Security Analysis: Professional security vulnerability scanner for Python code (checks for SQL injection, hardcoded passwords, unsafe functions, etc.)
- Secrets Detection: Scans for hardcoded secrets, API keys, passwords, and tokens using detect-secrets
- Code Quality (Flake8): Python code quality and style checker.
- File Permissions: Checks for files with executable or unusual permissions
- Suspicious Files: Detects suspicious file types, hidden files, or unexpected executables

The result is detailed under the Plugin Version Details Page > Security Scan tab.
More details about these are available at https://plugins.qgis.org/docs/security-scanning

The proposition for now is to make the **Bandit Security Analysis** and **Secrets Detection** a blocking validator if at least one critical issue is found. When critical issues are detected, the plugin version will be blocked from approval and public download until the issues are resolved.


## Bandit Security Analysis

Critical vulnerabilities found here block the plugin from approval and download until fixed. Email includes:

- Issue summary and severity
- File/line locations
- Remediation guidance

![](./images/qep408/bandit-check.png)


## Secrets Detection

Hardcoded secrets block the plugin until removed. Email includes:

- Secret type (API key, password, token, etc.)
- File/line locations
- How to fix it

![](./images/qep408/secrets-check.png)

## Other checks

Code quality, file permissions, and suspicious files checks 

![](./images/qep408/other-checks.png)

## Asynchrounous run

All the checks mentioned above will run asynchronously after the plugin upload and their results are included in email notifications and displayed on the Plugin Version Details Page > Security Scan tab. These comprehensive checks provide maintainers with actionable feedback on code quality and potential issues to address in future updates.


## Risks

Plugins aren't available during the check (usually a few minutes). This is mitigated by:

- Fast validation (most checks complete in minutes)
- Immediate email notifications
- Clear status indicators ("Validating" or "Blocked")
- Easy re-upload workflow
- No vulnerable plugins ever reach users

This balances fast uploads with strong security guarantees.

## Performance Implications

- Uploads complete instantly (no waiting for checks)
- All checks run in parallel via task queue
- Better server resource management
- Immediate upload confirmation for developers

## Further Considerations/Improvements

### Asynchronous Validation Architecture

The implementation uses a task queue-based architecture to process all checks asynchronously:

**Upload Flow:**
1. Plugin uploaded and stored
2. Status set to `validating` (not yet available)
3. Validation task queued
4. Instant confirmation email to maintainer
5. All checks run in parallel
6. Results stored in database
7. Status updated:
   - `validated` (no critical issues) → Available for approval/download
   - For trusted users: auto-approved if checks pass
   - `blocked` (critical issues) → Unavailable until fixed
8. Results email sent to maintainer (and approvers if ready for review)

**Components:**
- **Task Queue**: Celery or similar task broker for asynchronous job processing
- **Workers**: Background processes that execute validation checks in parallel
- **Database**: Stores validation results, check status, and issue details
- **Email Service**: Sends notifications at key stages (upload confirmation, results)

### Email Notifications for Validation Results

Email notifications are sent at two key stages:

#### Stage 1: Upload Confirmation (Immediate)

Sent immediately upon successful upload to confirm receipt:

**Subject:** Plugin Upload Confirmation: [Plugin Name] v[Version]

**Content:**
- Plugin name and version
- Upload timestamp
- Validation status (Validating)
- **Important notice**: Plugin is not yet available for approval or download
- Estimated completion time
- Link to track progress on plugin details page
- What to expect in the next email

#### Stage 2: Validation Results (After Checks Complete)

**Subject:** Plugin Validation Results: [Plugin Name] v[Version]

**Content for Clean Validation (All Checks Passed):**
- ✓ All checks passed
- **Plugin status**:
  - Trusted users: Auto-approved and available for download
  - Regular users: Ready for approval (approvers are notified)
- Summary of checks performed
- Link to detailed results

**Recipients:** Plugin maintainer(s) + Plugin approvers (when ready for review)

**Content for Issues Found:**
- Issues summary (count by severity)
- **Availability status**:
  - Critical issues: **BLOCKED** - not available until fixed
  - Non-critical only: Available for approval, but improvements recommended
- Critical issues details:
  - Bandit/Secrets Detection findings
  - File/line references
  - How to fix
- Non-critical issues:
  - Code quality suggestions
  - Optional improvements
- Link to re-upload corrected version
- Support contact

**Recipients:** Plugin maintainer(s)

**For each issue type in email:**
- Clear title and severity badge
- Specific details (file names, line numbers, issue descriptions)
- Example fixes or remediation guidance
- Best practices documentation links

### Best Practices Documentation

Emails link to the (docs)[https://plugins.qgis.org/docs/security-scanning] on:
- Fixing Bandit security issues
- Secret management
- Code quality (Flake8)
- File permissions
- Handling suspicious files

### Security Considerations

- Emails contain no sensitive data beyond what maintainers need to fix issues
- No API keys or passwords are included in emails
- Validation reports are also available via secure web interface
- Email access logs are maintained for audit purposes
- **Plugins with critical security issues are never made publicly available**, protecting end users from vulnerabilities

### Future Enhancements

As the security and QA validation system matures, additional checks are planned to be added over time, all following the same asynchronous pattern:

- **SPDX License Header Requirements**: Validation of proper license headers in source files
- **Binary Blob Detection**: Identification of unexpected binary files that may pose security risks
- **GPL Compliance Checks**: Verification of GPL license compliance and proper attribution
- **Additional Code Quality Metrics**: Extended static analysis and code quality validations
- **Dependency Vulnerability Scanning**: Checking for known vulnerabilities in plugin dependencies
- **Malware Signature Detection**: Advanced scanning for known malicious patterns

All new checks will automatically integrate into the existing asynchronous validation pipeline and be included in validation result emails.


## Issue Tracking ID(s)

https://github.com/qgis/QGIS-Plugins-Website/issues/213
