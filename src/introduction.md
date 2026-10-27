# Introduction

Model capability is not research capability. A capable model can still produce weak vulnerability research when it lacks target context, deterministic tools, authorization boundaries, durable state, and a validation method.

In this workshop, you will compare a generic agent response with an investigation performed through a target-specific harness. You will preserve a baseline, select and scope an authorized open-source target, build reusable research context and procedures, and test one specific security hypothesis when the target can run safely.

You will assemble a harness around your selected target, not run a bundled scanner or ready-made research scaffold. Read [Local Validation Environment](harness-components/local-validation-environment/index.md) to prepare an isolated workspace. The lessons supply their necessary instructions and examples; no trainer’s personal tooling or local directory is required.

Agent output is a lead, not a finding. You remain responsible for authorization, security judgment, runtime validation, impact analysis, and responsible disclosure. Test only local systems, isolated environments, or systems for which you have explicit authorization. Do not test public project infrastructure or publish suspected unpatched vulnerabilities.

A vulnerability is not a promised outcome. The intended result is a working harness, evidence about one investigated hypothesis, and a repeatable method for continuing the research.
