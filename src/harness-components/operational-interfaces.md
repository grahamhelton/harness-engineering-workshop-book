# Operational Interfaces

A vulnerability research harness can be hard to operate and observe. Two common examples are:

1. Starting a target may require several commands and configuration choices.
2. Checking what an agent is doing may mean searching through its output or opening files one by one.

Both people and agents can lose time reconstructing how to do a routine task or where to look next. From the agent's perspective, this has context window implications. If a task is performed often, the harness should expose an interface for the agent to perform that task without it having to re-identify the correct sequence of actions to take each time.

An **operational interface** gives them a clear way to perform an action or inspect the work without handling every step underneath. This lesson looks at named commands for operating a target and different views for inspecting work. The examples are concepts and tools taken from some of my workflows, not tools every project needs.

## Why interfaces help

Operational interfaces provide clear ways to start work, see what is happening, guide the agent, and/or review its results. This is analogous to a steering wheel and dashboard in a car; they let you control and understand the work without handling every underlying component directly.

Without an interface, each interaction starts with a small discovery task such as:

- Which command starts the target?
- Which output matters?
- What is the status of current work?

A good interface should make the intended way to perform a task more visible. Both the researcher and agent should be able to work out how to do something by exploring the project structure, but repeating that process makes work slower and presents more opportunities to get things wrong.

From a token optimization perspective, using interfaces can also reduce what an agent has to keep in its immediate context window. It can run a known command or inspect the current work instead of spending tokens rethinking how to do a common task like starting a dev server.

## Controls: named commands

Named commands give people and agents a consistent way to perform recurring tasks.

If you or an agent repeatedly have to look up a sequence of commands, paths, or configuration choices to do the same task, that task is a good candidate for a named command. If it takes several steps to start a target in your specific environment, those steps should be codified into some sort of interface.

For example, a Makefile or [justfile](https://just.systems/man/en/) could provide a single command that starts the required services with the right configuration. People and agents can then run the same command without reconstructing the setup from memory.

For a local Kubernetes research target, a justfile can provide a few named commands for starting the lab and observing its behavior:

```just
cluster := "research-lab"
node := cluster + "-control-plane"

# Create an isolated Kubernetes cluster with audit logging enabled.
lab-up:
    kind create cluster --name {{cluster}} --config lab/kind.yaml

# Show whether the cluster is running.
lab-status:
    kubectl --context kind-{{cluster}} get nodes

# Stream the API server audit log.
audit-tail:
    docker exec {{node}} tail -F /var/log/kubernetes/audit.log
```

This gives both humans and agents a consistent way to run tools like `just audit-tail` and start streaming logs from a Kubernetes cluster.

The commands also keep project-specific details in one place. If the cluster name or log location changes, there is one centralized place that does not require remembering the new details.

## Views: inspecting work

Named commands are useful when you know what action you need to perform whereas a **view** helps when you need to understand what is happening. If you repeatedly piece together output just to find the status of a piece of work, consider a view that presents it directly.

This is analogous to a view in a database. It distills a vast set of data into only what you need at this moment.

While a view can easily (and often should be) output to a terminal, it doesn't have to be. It can be very helpful to have views that are easy for a human to read in the browser and an agent to read as text in a terminal.

For example, [Git-bug](https://github.com/git-bug/git-bug) lets a researcher open an issue in a browser to inspect its status and history. Here the researcher has opened an example issue in Git-bug's WebUI:

![Git-bug issue detail page for the example single-writer investigation.](../assets/operational-interfaces/Pasted%20image%2020261022084208.png)

Here, the researcher asks an agent for the issue’s status. The agent finds and opens the issue through a command rather than the browser. During an investigation, it could use that same command to check existing work before deciding what to examine next.

![Agent using a Git-bug skill and commands to check the same issue's status.](../assets/operational-interfaces/Pasted%20image%2020261022084052.png)

In this example, an agent loads a skill that explains how to use Git-bug, lists the issues, and opens the matching issue before reporting its status. Both the researcher and the agent use different interfaces to inspect the same issue.

It's important to make sure your views are not creating "new" data, as this can produce conflicting information that will cause confusion. When considering a new view, ask, "Will it expose existing work, or create a second copy that can disagree with it?"

You do not need a named command or view/dashboard for every part of a harness. Look for the actions you repeatedly reconstruct and the information you repeatedly struggle to find and give those tasks a clear interface to interact with. When people and agents use different interfaces, make sure they operate on the same underlying data.
