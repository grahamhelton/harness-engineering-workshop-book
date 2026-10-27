# Skills

A skill is a reusable set of instructions that give an agent additional guidance for a particular kind of work. Generally, skills are useful when there is a particular way you want to accomplish a task. They give agents a method for a recurring task instead of requiring the model to reconstruct that method during every session. If you find yourself in a situation where you're instructing a model how to do a specific task repeatedly, that may be an indication you should write a skill for the process.

Skills improve procedural consistency, but do *not* provide determinism. The model still interprets the procedures, chooses which actions to follow (or not), and evaluates observations.

At a high level, a skill is just a folder containing a `SKILL.md` file. It can optionally contain other files, such as:

```text
report-research-progress/
├── SKILL.md                 # When and how to update an investigation
├── references/
│   └── reporting-tool.md    # Optional: Reporting operations and their fields
└── assets/
    └── update-template.md   # Optional: Format for a progress update
```

A good `SKILL.md` should follow [a specific format](https://agentskills.io/specification) defined by the spec.

| Field           | Required | Constraints                                                                                                       |
| --------------- | -------- | ----------------------------------------------------------------------------------------------------------------- |
| `name`          | Yes      | Max 64 characters. Lowercase letters, numbers, and hyphens only. Must not start or end with a hyphen.             |
| `description`   | Yes      | Max 1024 characters. Non-empty. Describes what the skill does and when to use it.                                 |
| `license`       | No       | License name or reference to a bundled license file.                                                              |
| `compatibility` | No       | Max 500 characters. Indicates environment requirements (intended product, system packages, network access, etc.). |
| `metadata`      | No       | Arbitrary key-value mapping for additional metadata (a map from string keys to string values).                    |
| `allowed-tools` | No       | Space-separated string of pre-approved tools the skill may use. (Experimental)                                    |

```markdown
---
name: report-research-progress
description: Use when evidence changes an investigation's assessment, status, blocker, or next action. Guide the agent in sending an accurate update through the research reporting tool.
---
# How to report findings
`<Insert markdown instructions here>`
```

The name identifies the skill to the agent, and most harnesses also let you invoke it manually by typing this name. The description is what helps the agent decide when to use the skill. If you notice your skill is getting loaded at the wrong times, you should create a more refined description. Below the frontmatter, the Markdown body contains the instructions the agent follows once the skill is loaded.

## How a skill enters the agent loop

Agents do not read every skill in full at the start of a session. This is harness dependent, but generally the agent can see each skill's name and description. When a task matches the description, the harness makes its full instructions available to the agent. The agent then uses those instructions while working on the task.

## Skills and other harness components

A skill is useful for providing markdown instructions to an agent, but tools are still what perform the actions defined in the skill's markdown instructions. In the above example about reporting, the skill may tell an agent that an update is warranted, what evidence to include, how to format the report, and what tools to use to actually write the report.

The agent could then use a CLI tool or MCP server to post the update to a project tracking tool like Linear or Jira. The skill itself does not add the reporting capability; the relevant tools must be available and the agent must be configured to use them.

## Vulnerability research example

During vulnerability research, an agent can surface a plausible-sounding vulnerability but leave the researcher without enough information to manually verify it. When assessing the claim, we need to know *at least* the relevant code paths, the hypothesis that was tested, test conditions, version numbers tested, threat model, and evidence for (and against) the claim. Reconstructing that context takes a long time.

When an agent surfaces a vulnerability, a skill provides the agent with a set of instructions for how to package up the suspected finding for manual review. It should specify which versions, source paths, evidence, assumptions, and any unresolved questions to include. The agent can gather this evidence and create a handoff bundle (I like having it create a tar.gz file) that we can move out of the harness and inspect independently or share with a wider team.
