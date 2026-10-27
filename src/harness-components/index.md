# Harness Components

A harness combines the context, tools, controls, state, and workflow around an agent. Add a component only when it solves an observed research problem.

The component lessons cover:

- a local validation environment for repeatable, isolated runtime checks;
- research state for durable hypotheses, evidence, decisions, and next actions;
- context management for selecting current, relevant, and authoritative information;
- hooks for deterministic behavior at agent lifecycle events;
- reusable skills for repeatable research procedures;
- deterministic tools for reproducible analysis;
- operational interfaces for stable local commands; and
- external integrations when local files and commands are not sufficient.

Not every harness needs every component. Add a component only when it addresses an observed research problem.

## How information moves through a harness

Information enters a harness through human input, target files, tool results, agent responses, and harness events. The harness places some of that information in the agent's current context.

Capture, persistence, and reuse are separate decisions. Information can be used and discarded, or it can be saved without being shown to another agent. To serve as memory, stored information must be selected and organized for future use.

The harness can supply memory automatically or let an agent retrieve it when needed. Once retrieved, memory returns to the current context and can influence new work, which may produce another update.
