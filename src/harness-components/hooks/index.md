# Hooks

In July 2026, OpenAI was running an evaluation on new models. The agents exploited shared infrastructure, obtained external access, and eventually attacked (and compromised) Hugging Face. There were a **number** of embarrassing security failures; however, the incident highlights the need to have some level of visibility into what actions are being performed by agents.

Relying on the model to follow instructions is not sufficient since the model is non-deterministic. One such way to observe an agent's activity is by "hooking" into the agent during its lifecycle.

A hook lets the harness invoke pre-configured behavior at a specific lifecycle event. For example, when a harness runs a tool call, a hook may be configured to log or block the tool invocation.

Before discussing hooks, it would be helpful to understand *what* we're hooking into. When working with agents, there are two loops that you should be familiar with when working with hooks: the Session Lifecycle and the Agent Loop.

![A session starts or resumes, contains a first turn and later turns, then ends. The first turn shows the user submitting input, the harness preparing context and invoking the model, a model tool request passing through a pre-tool hook and permission check, tool execution and a result returning to the harness, a post-tool hook, another model invocation, and the response returning control to the user. Hook event names are shown at the corresponding boundaries.](session-lifecycle-agent-loop.png)


## The Session Lifecycle

A session is a harness-managed interaction that carries state across one or more agent turns. A harness may expose a lifecycle event when:
- A session starts or resumes
- Input is submitted
- A turn stops
- Context is compacted
- A session ends

Session events are useful places for hooks because they occur regardless of whether the model remembers to perform an action. For example, a session-start hook can deterministically load current research scope before the first model invocation, while a pre-compaction hook can preserve information before context is reduced.
## The Agent Loop

To understand hooks, we must first understand the agent loop. The agent loop is described in slightly different ways depending on whose documentation you're reading. [OpenAI](https://openai.com/index/unrolling-the-codex-agent-loop/) describes the agent loop as the harness logic that orchestrates interaction between the user, model, and tools. A single turn can contain repeated model invocations and tool calls before control returns to the user, whereas [Anthropic](https://www.anthropic.com/engineering/building-effective-agents#agents) describes agents as models using tools and environmental feedback in a loop with human checkpoints.

For our purposes, either conceptual model is fine. The important part is that at a high level a harness exposes points in this loop as lifecycle *events*.

```text
User configures behavior
        ↓
Harness reaches a lifecycle event
        ↓
Harness invokes the configured handler
        ↓
Handler observes or affects the event
```

Some hooks that run inside the agent loop are:

| Hook               | Scope             | Frequency                                       |
| ------------------ | ----------------- | ----------------------------------------------- |
| `SessionStart`     | Session lifecycle | At session start, resume, or similar transition |
| `UserPromptSubmit` | Turn boundary     | Usually once for each submitted prompt          |
| `PreToolUse`       | Agent loop        | Before each matching tool call                  |
| `PostToolUse`      | Agent loop        | After each matching tool call                   |
| `Stop`             | Turn boundary     | When the agent attempts to finish               |
| `PreCompact`       | Session lifecycle | When context compaction is about to occur       |
| `SessionEnd`       | Session lifecycle | When the harness considers the session ended    |
# Git analog

A hook in the agentic loop is analogous to a [git hook](https://git-scm.com/book/ms/v2/Customizing-Git-Git-Hooks). For example, when a user runs `git commit`, Git can invoke a `pre-commit` hook before creating the commit and a `post-commit` hook after creating it. The user configures the hook, but it only fires at a specific point in the Git lifecycle. An agent harness works similarly. The harness exposes lifecycle events and invokes configured handlers when those events occur. For example, it might run one hook before a tool executes and another after the tool runs.
