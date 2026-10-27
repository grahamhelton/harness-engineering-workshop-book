# Problems in Agent-Assisted Vulnerability Research

You did not train the models you use for research. You inherit their behavior, including how they interpret a security question and what they consider a complete answer. The agent's tools and initial instructions also shape that first result.

## Claims exceed the evidence

An agent can identify suspicious code and overstate impact before it establishes the conditions needed for an attack.

Suppose an agent reports that an attacker can change a user's email address through [Cross-Site Request Forgery (CSRF)](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html). The agent found no `Origin` header check in the account settings handler. That source observation does not show whether an attacker-controlled page can cause the user's browser to submit the change successfully. A successful request from a terminal leaves that browser behavior untested.

A Common Vulnerability Scoring System (CVSS) score can look precise even when its inputs are guesses. The [CVSS v4.0 calculation](https://www.first.org/cvss/v4.0/specification-document) does not validate the assumptions behind those inputs.

These reporting problems predate language models. A dependency can contain vulnerable code that the application never calls, yet its presence can still appear as a finding in a report.

An untested claim leaves the maintainer with the missing research, regardless of how complete the report looks.

## Research standards stay implicit

A general instruction to be accurate leaves much of the research process unspecified. The agent may lack the project's documentation, examples of sound work, or records of previous pitfalls. It may choose tools that cannot answer the research question, even while it appears to follow the instruction.

## Validation can stop with unanswered questions

A model may suggest severe impact but refuse a step needed to check whether the claim holds. A failed test setup can leave the same gap. The investigation stops with an unresolved claim, and the report may not make that limit clear.

## A different answer can look like better research

A longer report, a lower severity score, or a more cautious tone does not by itself show that the research improved. The change needs a basis you can inspect.

## How we will approach these problems

Harness engineering gives you ways to shift more of the research process toward your standards. You make the method explicit, supply the information and tools it needs, and inspect whether the agent follows it.

The [component lessons](harness-components/index.md) will address these needs through a usable test environment, external information, saved research, context selection, and operational interfaces. Documentation, examples, and records of previous pitfalls give the agent ways to apply your standards. Written instructions alone do not establish that it followed them; the actions and retained evidence must show that.

We will use an empirical approach that keeps observations, interpretations, and untested hypotheses distinct. Each claim needs evidence that supports it under the stated conditions. When evidence contradicts a claim, revise or reject it under those conditions. When validation stops, retain the blocker, the unanswered question, and a permitted next action. Some blockers, including model refusals, may remain unresolved.

Use your selected, authorized project for an initial broad request for vulnerability analysis. Save the request and response, the source revision, and the checks the agent actually performed. This is your baseline.

Choose one claim to examine. Where is its supporting evidence? Which conditions did the agent assume? What check could support or reject it? If the result contains no useful lead, you can select a security boundary to investigate without inventing a vulnerability.

Keep any strengths in the baseline. Compare the later work with the original evidence, uncertainty, and effort; preserve ties and additions that do not help. The comparison gives you a reason to keep or change a part of the harness. Your judgment still determines what the evidence establishes and whether a finding is ready for disclosure.

Look for explicit uncertainty, specific gaps, and evidence for each claim.
