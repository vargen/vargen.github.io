---
layout: default
title: Findings
parent: CI Fuzz
nav_order: 5
permalink: ci-fuzz-findings
---
# **Findings**
{: .no_toc }

This section contains information about findings in CI Fuzz. How to access them, how they are categorized, and ways CI Fuzz can help with managing them.

## Table of contents
{: .no_toc .text-delta }

- TOC
{:toc}

---

## Findings Overview

Findings for a specific project can be viewed by clicking on **Findings** on the left sidebar. For a given run, CI Fuzz will display the total number of findings and the quantities for each severity (Cricital, High, Medium, and Low). These can be selected to filter for the findings you are most interested in viewing. There may also be categories depending on the specific findings that were discovered during the fuzz run.

![](/assets/images/ci-fuzz-findings/ci-fuzz-findings-categories-filters.png)

## Severity

Each finding discovered by CI Fuzz will have a `severity score`, a numerical score in the range 0.1 - 10.0 and an associated classification. The `severity score` and `severity classification` use the same values as [CVSS](https://www.first.org/cvss/specification-document) to enable you to easily integrate with other processes and tools that recognize this standard.

| Severity Score    | Severity Classification   |
|:------------------|:--------------------------|
| 0.1 - 3.9     	| Low                       |
| 4.0 - 6.9   	    | Medium                    |
| 7.0 - 8.9	        | High                      |
| 9.0 - 10.0        | Critical                  |

### Source of Severity
{: .no_toc }

The `severity score` and `severity classification` provided by CI Fuzz represent the importance of a finding. This score is based on the nature of the bug or vulnerability in a generic context. The score does not account for the specifics of a given vulnerability such as the number of versions affected, ease of exploitation, etc. 

### Finding Categories
{: .no_toc }

Findings may have additional categories indicating it belongs to a group of findings. These categories are:

* OWASP - the finding is included in the [OWASP Top 10](https://owasp.org/www-project-top-ten/).
* External - the finding using the fuzzer input was confirmed by ZAP. These findings, since they are based solely off the input and not triggered by the fuzzer, will not contain a detailed stack trace or line numbers.
* Regression - the finding had been marked as fixed but has now reoccurred. False positives may occur if not all tests have been executed in the current or previous run or if tests have been modified.
* API - A finding with an “API” category was found by a Web API fuzzing test. Depending on the availability of the API under test, this type of issue may be one that can be readily exploitable by anyone with the same access. 

## Managing Findings

CI Fuzz provides capabilities to help you manage your findings so you can properly triage, track, and ultimately fix them.

### Findings Overview

There are three you can see an overview of current findings for a given project:

* The Dashboard will contain the available project cards. This can be found in the left pane near the top.
* Select the project from the dropdown menu in the left pane (just below **Dashboard**) and then click either **Overview** or **Findings** in the left pane.

The overview provides a clear view of:

* The current number of findings, total and new.
* The `severity classification` of these findings.
* The `categories` associated with the findings.

![](/assets/images/ci-fuzz-findings/ci-fuzz-findings-overview.png)

You can click on the `severity classification` or one of the `categories` to quickly filter for the findings you consider highest priority.

Clicking on one of the filters or on the project pane itself will show you the list of findings. Click a specific finding to obtain additional information.

![](/assets/images/ci-fuzz-findings/ci-fuzz-findings-lower-pane.png)

### Finding Status

A finding can have 1 of 3 different states: `Open`, `Assessed`, or `Ignored`. You can adjust these by selecting the `Action` dropdown on the right side.

* An `open` finding indicates this finding has not been processed in any way. If the fuzzer discovers this finding in the future, it will be reported again.
* An `assessed` finding is one that is currently being analyzed. Assessed findings prevent the fuzzing run from failing when they are rediscovered, but will still alert the user they were found.
* `ignored` status is for any findings you consider should not be fixed for whatever reason. The fuzzer will not report this finding if it encounters it again.

#### **Type and ID**
{: .no_toc }

The type of the finding (Stack Buffer Overflow, Sql Injection, etc...) and an identifier used by `CI Fuzz`. 

#### **Location**
{: .no_toc }

This column will contain the location where the fuzzer discovered the finding.

#### **Download Findings**
{: .no_toc }

You can download the findings as a pdf, Word document, or Excel document. Click on the ![](/assets/images/ci-fuzz-findings/ci-fuzz-download-findings.png) that is located just above the actual findings.

#### **Link to Ticket System**
{: .no_toc }

CI Fuzz can link to external ticket systems to help manage the remediation process for a finding. Under the `Action` menu on the right of the finding, click ![](/assets/images/ci-fuzz-findings/ci-fuzz-link-issue.png), paste the URL of the created issue (from Jira, GitHub, etc...) and click Link.

### Finding Details

When you expand a finding in the bottom pane (by clicking the `>` on the left side of a finding), it will contain 3 tabs with different information: `Debug`, `Description`, and `Log`. These tabs contain several pieces of information that can help you determine the root cause of the finding. 

* The `debug` tab contains the fuzz test responsible for the finding, the source line, the stack trace (if available), and the crashing input. 
* The `description` tab contains the `severity score`, a short description of the finding, and possibly some links to additional information about this type of finding.
* The `log` tab content depends on the type of fuzzing that discovered the finding. If the finding is from unit fuzzing, then the output will be output directly from the fuzzer. If the finding is from Web API fuzzing, then the output will contain the API request responsible for triggering the finding.


