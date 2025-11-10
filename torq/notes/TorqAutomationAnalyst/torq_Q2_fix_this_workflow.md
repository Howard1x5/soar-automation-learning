Torq Automation Analyst – Fix This Workflow Lab
1. Summary

This practical exam from Torq Academy – Automation Analyst provided a broken YAML workflow titled “Fix This Workflow.”
The objective was to import the workflow, identify the misconfigurations preventing it from producing valid output, correct the logic, and recover the hidden quiz key embedded in the Python step.

Although Torq SaaS access wasn’t available, the YAML file (Q2_fix_this_workflow.yaml) was reverse-engineered locally to analyze structure, logic flow, and the encoded key.

2. Objective

Repair the workflow so that:

All CVE data loops and collectors function correctly.

Severity cases map properly (Critical, High, Medium, Low).

The output returns a total of 50 results.

The Python quiz step validates successfully and reveals the encoded answer.

3. Approach

Tools used

Local text editor & YAML linter for syntax validation.

Manual inspection and pattern search for {{ steps.* }} and base64 strings.

Logical tracing of Jinja variables, step IDs, and output references.

Methodology

Parsed the YAML into structural blocks: trigger → loop → switch → collectors → Python quiz.

Mapped all data references ({{ $.something }}) to actual fields returned by the NVD CVE API.

Identified base64-encoded strings and decoded them manually to reveal the final quiz result.

Validated logic by confirming variable consistency and expected loop counts.

4. Key Issues Identified and Fixes
#	Problem	Root Cause	Fix
1	Loop referencing wrong dataset	YAML looped over $.CVE_Items which doesn’t exist. NVD returns result.CVE_Items.	Changed to in: '{{ $.result.CVE_Items }}'.
2	Critical severity mismatch	Switch case compared rvalue: Critical (mixed case). API returns CRITICAL.	Changed to rvalue: CRITICAL.
3	High severity collector stored wrong field	Expression pulled references.reference_data instead of CVE ID.	Changed to expression: '{{ $.cve_value.cve.CVE_data_meta.ID }}'.
4	Medium collector overwrote valid results	Used operator NOT_EMPTY to clear results instead of when empty.	Changed to IS_EMPTY.
5	Optional cleanup	Date format used :000 instead of .000 milliseconds.	Cosmetic fix only.
6	Minor mismatch in Prepare Output step	Used .results instead of .result.	Updated to maintain consistency.