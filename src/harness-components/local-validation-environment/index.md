# Local Validation Environment

A local validation environment gives your agent a known way to run the target
application and use the tools it needs. The harness supplies operating standards
and checks that let you inspect whether the agent followed them.

## The problem

An agent may report that an application started when only a container was created.
A tool may be installed but unable to reach the selected engine or application.
Without clear checks, later research can depend on a setup that never worked.

Start with the project's normal setup. Give the agent enough information to run
the application inside the approved boundary, establish a small usable toolset,
and verify the result. Its report should point to observed output, not just a
statement that setup is complete.

## Define what working means

Decide what evidence would establish these conditions for your project:

- The intended application is running and reachable from the agent's tools.
- The tools needed for the next task are available and functional.
- A harmless normal operation produces the expected result.

For a web application, that normal operation might be a documented health check
or a request for a known page. Choose a check that identifies the application
you intended to start. A response from an unrelated service on the same port
would not establish readiness.

Keep the toolset small. A container runtime such as Podman may run the target;
curl may let the agent make requests. Your project may use different tools.
Each tool should answer a question or perform an operation the agent actually needs.

## Give the agent standards it can check

Put project-specific operating guidance where your agent can read it, such as
an `AGENTS.md` or an equivalent instruction file. Decide what the agent needs
to know about:

- The approved runtime, project setup instructions, and permitted setup actions.
- The tools required for the task and how to check that they work.
- The expected readiness result and where setup output should be retained.
- The allowed filesystem and network boundary, credentials, and owned resources.
- Conditions that require the agent to stop and report a blocker.

You choose the wording and level of detail. If your agent does not load the
file automatically, make reading it part of the task. The useful question is
whether its actions follow the standards, not whether it can repeat their text.

Instructions describe the boundary. Runtime isolation, tool permissions, and other
execution controls enforce the limits available in your environment. Keep those
controls in place; a successful setup is not permission to expand the task.

## Ask for observable checks

The agent can check tool availability with commands such as:

```sh
command -v podman curl
```

This finds command paths. It does not establish that an engine is reachable or
the application works. What further check would distinguish those conditions
for your project?

Look at the actions and outputs your agent retains. Did it use the approved
runtime and project setup? Did it verify the application with a normal operation?
Can you identify the resources it created and how to stop them safely?

When a tool is missing, an engine is unavailable, or setup fails, the agent
should preserve the failure and report the next permitted step. It should not
switch to public infrastructure, use unrelated credentials, or alter another
environment to obtain a successful result.

## Apply this to your project

Choose the smallest setup and toolset that can support your next research task.
Use your agent to perform the permitted setup and readiness checks. Consider:

- Which existing project instructions can the agent use?
- What evidence would show that setup met your standards?
- Which assumptions about tools or application readiness still need a check?
- When should the agent ask for help instead of changing the environment?

If you revise the guidance, retain the earlier result so you can inspect what
changed. Did the agent's behavior improve, or did the added instructions create
work without a useful difference?

Your checkpoint is a setup record: the application and tools used, readiness
evidence, owned resources, and one justified next action. A precise blocker is
also a useful result. Keep evidence separate from disposable application data,
and stop only resources you own and are authorized to stop.

## Extend the environment

More detailed research may need known accounts, permissions, and application data.
For an access-control investigation, your setup could prepare two users, an
administrator, and resources with known owners. The agent would verify those
conditions before tests begin. A permission table based on the project's documented
rules would supply the expected behavior. Compare responses and actual state
changes with those rules; an HTTP status alone cannot establish whether access
controls worked.

An application with several services may also need database migrations, a queue,
and a worker. Readiness could include a normal operation that depends on those
services. Setup and reset procedures would need to preserve evidence while they
restore known application state.

A [Nix flake](https://nix.dev/concepts/flakes.html) and devshell can define the
compiler, runtime, and research tools with locked dependency versions. This helps
another researcher recreate the tool environment on supported platforms. Application
data, credentials, and isolation still need separate controls.

These are possible extensions of the basic setup. Add the conditions your next
research question needs, and give the agent a procedure to establish and verify
them. Each addition also needs maintenance and checks of its own.
