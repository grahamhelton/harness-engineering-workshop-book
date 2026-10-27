# Context Management

Context management is deciding what information an agent receives for its current task. That information can include instructions, the conversation, files, and tool results. The agent does not need every available file or tool result for every task.
## The problem

Vulnerability research produces more information than an agent needs for any one decision. At any given moment there may be source code, logs, telemetry, tool results, and research notes available to the agent. Most of it may be useful at some point, but only some of it is relevant to the task the agent is working on now, at this moment.

This creates two problems. If the agent lacks necessary information, it cannot assess the question properly. If it receives too much irrelevant information, that material can influence its answer even when the necessary information is present.
## What is the context window?

The context window is the information the model can use when producing its next response. It includes the current request and any instructions, conversation history, files, or tool results supplied to it. A file the agent could open is not in the context window until its contents are read.

The context window is very important in vulnerability research because the agent's assessment depends on the evidence it can examine. It cannot inspect evidence that has not been supplied to it, and the amount of information it can receive is limited.

## Context window selection

Selection means choosing which available information to put in the agent's context for the current task. This means which files to open, which tool results to inspect, and what parts of earlier work to bring back into context.

Selection also affects what the agent treats as important. A long tool result can quickly fill up a context window with irrelevant information. The goal is to put enough relevant information in the context window for the agent to complete its task.

As an example: Imagine the task is to find the current temperature in San Francisco. This request asks Open-Meteo for the current temperature and 16 days of hourly data.

```sh
curl -fsSL 'https://api.open-meteo.com/v1/forecast?latitude=37.77&longitude=-122.42&current=temperature_2m&hourly=temperature_2m,relative_humidity_2m,precipitation_probability&forecast_days=16'
```

Now run the same request, but select the fields needed for the question before the output reaches the agent:

```sh
curl -fsSL 'https://api.open-meteo.com/v1/forecast?latitude=37.77&longitude=-122.42&current=temperature_2m&hourly=temperature_2m,relative_humidity_2m,precipitation_probability&forecast_days=16' | jq -c '{temperature: .current.temperature_2m, unit: .current_units.temperature_2m}'
```

When I ran these commands, the unfiltered response was about 12 KB. The filtered output was only 34 bytes: `{"temperature":22.2,"unit":"°C"}`. Both contained the current temperature and its unit, but the second filtered out the unnecessary data before it reached the context window.

The weather example shows why we might leave information out, but we also need to decide what must be included. For vulnerability research, that can include the project's instruction files and system prompts.

## Instructions in context

Some instructions are loaded into the context window before the agent begins work. In vulnerability research, these instructions can define the scope of an investigation, how to handle suspected findings, and which [operational interfaces](../operational-interfaces.md) and [deterministic tools](../deterministic-tools.md) the project provides. They can also inform the agent on how to use the [local validation environment](../local-validation-environment/index.md) instead of leaving it to the model to figure out on its own every time.

A **system prompt** is a set of instructions the coding agent supplies to the model when it starts a session. An `AGENTS.md` is a file containing instructions for work in a particular project. The coding agent loads that file into the agent's context alongside its system prompt.

These instructions differ in where they are set. You can change a coding agent's system prompt through its configuration, while an `AGENTS.md` lives in the directory structure. Most coding agents load an `AGENTS.md` automatically, and system prompt instructions generally take precedence over lower-priority instructions, but how a coding agent loads project files determines how conflicts are handled.
