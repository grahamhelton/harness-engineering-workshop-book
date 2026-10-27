# Telemetry

Telemetry is information that the harness captures about agent and system activity. It might include submitted prompts, tool invocations, completion events, agent responses, timestamps, durations, and session identifiers. Telemetry should not be an agent's account of its own work; instead, telemetry should be deterministically recorded by the harness at the point where an event occurs.

Because telemetry sits outside the agent, it should be able to preserve activity after a conversation is compacted or deleted. This makes telemetry useful for reconstructing sessions, investigating failures, and understanding how the harness behaved.

Telemetry should record activity, not establish that an agent's conclusion is correct or that an observed behavior has security impact.

For example, an append-only JSON Lines ledger could contain this illustrative tool event:

```json
{"timestamp":"2026-10-08T19:12:03.008000+00:00","event":"PostToolUse","session_id":"abc123","turn_id":"turn-1","cwd":"/workspace/target","tool":"Bash","command":"python3 --version","duration_ms":21,"status":"completed"}
```

Because telemetry events are generally high-volume, these should almost never be loaded into an agent's context window. Where telemetry becomes useful is when you identify an area where the harness is failing and you're working on improving it.

## Why can't the harness just save everything?

Agents can create a lot of telemetry very quickly, and while you could create a system to save this, it's not all necessarily useful. The agent still has a finite context window that would quickly be overloaded if we dumped too much telemetry into it. This means we have to be selective about the information we are collecting and exposing to agents. To identify this information, it can be helpful to answer the following questions:

1. What is worth preserving?
2. At what level of detail should we preserve it?
3. How long does it remain valid?
4. Who needs to know about this?
5. When should it enter the context window?
6. When, if ever, should it be revised?
