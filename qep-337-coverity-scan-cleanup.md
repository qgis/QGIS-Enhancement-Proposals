# QGIS Enhancement: QGIS (and dependencies) Coverity Scan Cleanup 

**Date** 2025-03-25

**Author** Nyall Dawson (@nyalldawson)

**Contact** nyall.dawson@gmail.com

**Version** QGIS 3.44-3.46

# Summary

The [Coverity Scan](https://scan.coverity.com/) tool is a highly regarded tool for static analysis of complex
c++ projects. It's able to pro-actively identify many code issues (such as potential
crashes, memory leaks, and other unsafe behavior) which aren't picked up by other
tools (such as clang-tidy or cppcheck).

Having no open issues reported by Coverity Scan allows a project to advertise
themselves as "Coverity Clean", which is a highly regarded recognition and which forms
a valuable bullet point when discussing a project's stability and code quality.

While Coverity Scan is a proprietary service, it offers free analysis for
suitable open-source projects. It has been used sporadically for QGIS analysis
over the past 10 years, and has directly lead to many fixes and optimisations. 

Currently, the tool reports around 1075 open issues when run on the QGIS master codebase.

These issues range from false positives to trivial fixes through to serious issues
which require substantial work to fix. Unfortunately, the large number of trivial
issues currently reported in QGIS make the tool effectively useless for QGIS, as
the critical issues are hidden amongst the many hundreds of trivial issues.

This project will cleanup the Coverity Scan results by:

1. Marking false-positive issues accordingly in the scan results, so that they no
  longer clutter the Coverity results reports
2. Fix trivial issues so that the signal-to-noise ratio of the reports is improved

Unfortunately the Coverity Scan results site is very slow and clunky to navigate,
so even closing false-positive reports takes a non-negligable amount of time to do!

**Please note that Coverity Scan results are not publicly available due to the
potential of security risks being exposed, and only approved QGIS core developers
have access to these reports**

## External projects

Since QGIS embeds a number of third party dependencies (such as MDAL, lazperf, and untwine,
among others), the scan also includes issues identified in these projects.

During the cleanup I will submit pull requests upstream to these projects, which will
benefit both the QGIS scan results + other users of these libraries.

## Deliverables

- Removing all false-positive reports from the scan results
- Fixing trivial issues in QGIS and embedded third party dependencies

If time is available after completing the cleanup, I will investigate whether it is
possible to automatically run the Coverity Scan tool on a weekly basis as a GitHub
action (as is done for other projects, e.g. GDAL). **This may not be possible due to
the size of QGIS and the time required for a full QGIS rebuild under the Coverity Scan
tool.**

All non-trivial fixes are considered to be out of scope for the project. After the
cleanup these will be easy to identify and could potentially be targeted in a future
QGIS bug fixing round.
