# Memory

Good vulnerability research can be time-consuming and requires many iterations over notes, assumptions, logs, and source code to produce useful results. Memory preserves specific, high-value information, so a later researcher or agent can use it. Within a harness, memory is how we carry information about observations and outcomes into the next agent session.

Research state is one kind of memory: it preserves the current position of an investigation, with links to the observations behind it. Memory can also preserve useful target facts or procedures that are not part of a particular hypothesis. Saving either kind of information does not put it in a new agent's context; the agent still needs to find and inspect what is relevant.

## Problem

Coding agents like Claude Code and Codex are powerful, but they don't have a way of tracking and remembering details about an investigation:

Two common problems you may have run into when using coding agents for vulnerability research:
1. An agent maxes out its context window and is forced to compact the session, often losing key details about the investigation.
2. An interesting observation is made, but it is lost when the session is closed.

Imagine an agent tests whether a user with a viewer role can delete another user's report. The request returns a `403` Forbidden, and a separate check shows that the report still exists, then that session ends. When the session ends, a new session has no record of the request, response, or report status check. It does not know that this path was tested and what happened.

To make that test inspectable in a later session, the harness needs to do three things:
1. **Preservation**: Information needed later must survive the current session
2. **Discovery**: A later session must have a reliable way to find that information
3. **Retrieval**: The later session must be able to access and inspect that information

These three parts solve different problems. Preserving information does not help if a later session has no way of knowing it exists. Discovering information does not mean it is relevant to the current task. Retrieval makes the selected information available for inspection, but the agent still has to decide if it applies to the current task.

The viewer test shows how these requirements can work together.

The first session could preserve the request, the `403` response, the check that the report remained, and the conditions under which it ran (such as the target version and configuration). For discovery, it could leave a pointer in a location the next session is instructed to check. The next session that is working on a similar task could follow that pointer and retrieve the record for inspection.

This also helps cut down on reviewing findings. Having a ledger of work to reference during triage helps cut down on both manual and automated triaging.

## Hypothesis & Research state

A hypothesis is a specific, testable claim about security-relevant behavior. It serves as a seed, or a starting point for an investigation.

For example, a hypothesis may be "As a read-only viewer, I can delete a report that belongs to another user" or more narrowly "As a read-only viewer, `alice`, I can delete a report that belongs to `bob` by making a request to an API endpoint".

This is a testable claim that can produce evidence to support, narrow, or reject it.

**Research state** tracks the hypothesis, its current status, the evidence behind that status, important assumptions and environmental test conditions, rejected paths, and unresolved questions. It may also track the next step or when this claim should be reopened or reevaluated. Closed work also needs a condition for reopening it.

This differs from telemetry. One investigation into a hypothesis may create a large amount of telemetry, but it should only produce one useful update to the research state. Evidence should preserve only useful information to support a claim made in the research state.

For example, imagine an agent is testing the claim "As a read-only viewer, I can delete a report that belongs to another user". The agent may spin up a test environment, perform 100 HTTP requests, and find that one of them successfully deletes the report and receives a `200` response. It should not save every request sent as evidence. Instead, it should take the request/response that received a `200`, tear down the environment, and test it again under only those conditions. If it again receives a `200`, the research state should be updated with the evidence from the single atomic test.

Memory is complex. Start with a short markdown file for one check. Later, creating an index and separate evidence files becomes useful when multiple checks or sessions need to share an assessment. The format matters less than making the basis for a decision made in the research state inspectable.


## Example implementation

The example below uses markdown files to put preservation, discovery, and retrieval together.

A **thread** keeps related research work together across tests and sessions. In the viewer deletion example, the thread concerns whether someone with the viewer role can delete a report they should not control.

An **evidence record** preserves what happened in a particular check. In the first check, the deletion request returned `403 Forbidden`, and a separate check showed that the report remained. Those observations apply to the request and conditions tested.

The **thread index** is essentially an index of all the threads. It links to the evidence behind its assessment and records what remains unresolved. This is useful for context management when there are MANY open threads. Instead of reading each thread, the thread index can provide a single small file that can give an agent some info on each thread, and a pointer to the full evidence record which can be inspected only if it is needed.

One way to organize a thread is to give it a directory with a short `index.md` and separate evidence records:

```text
threads/
└── thr-001/
    ├── index.md
    └── evidence/
        └── E-001.md
```

The thread index describes the question, the current assessment, and one next action. Its evidence link lets a later session inspect the check behind that assessment.

`threads/thr-001/index.md`:

```markdown
---
id: thr-001
status: active
---

# Viewer deletion

## Question

Can a user with the `viewer` role delete a report they should not control?

## Current assessment

The tested deletion request did not delete the report. This does not
establish whether other deletion paths are available.

## Evidence

- [E-001: Viewer deletion attempt](evidence/E-001.md)

## Next action

Check whether another deletion path is reachable in the isolated target.
```

Following that link opens the evidence record for this check. It preserves the tested conditions, the server response, and the separate check of the report's state. Another test could change the index's assessment without changing what this record observed.

`threads/thr-001/evidence/E-001.md`:

````markdown
---
id: E-001
thread: thr-001
target_revision: example-revision
configuration: local-test
---

# Viewer deletion attempt

## Conditions

- Target: isolated local instance at `example-revision`, local-test configuration
- Actor: `viewer-a` with the `viewer` role
- Report: `42`, owned by `author-b`

## Request

```http
DELETE /api/reports/42 HTTP/1.1
Host: 127.0.0.1:8080
Authorization: Bearer [redacted]
```

## Observed response

```http
HTTP/1.1 403 Forbidden
Content-Type: application/json

{"error":"forbidden"}
```

## Target-state check

A follow-up `GET /api/reports/42` as `author-b` returned `200 OK`.
Report `42` remained present.
````
