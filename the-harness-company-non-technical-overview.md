# The Harness Company — What We Are Building and Why

## 1. The Simple Version

**The Harness Company** is building an open-source system that lets developers program, test, inspect, and modify their own AI agent workflows.

We are not trying to sell one fixed end-to-end AI workflow.

Instead, we are building the runtime that lets developers and teams assemble their own workflow from interchangeable parts.

The core idea is:

> Use our harness runtime to program, test, inspect, and modify your own agentic workflow.

Or, more directly:

> Program your agentic workflow with plugins, then run, inspect, replay, and improve it locally.

The planned domain is:

> theharnesscompany.io

---

## 2. What Problem Are We Solving?

AI agents are becoming more useful, but most agent workflows are still messy.

A developer may use one tool for planning, another for coding, another for testing, another for reviewing, and another for deployment. These pieces often do not work together cleanly.

Today, many agent workflows are:

- hard to inspect
- hard to debug
- hard to repeat
- hard to modify
- hard to trust
- hard to share
- hard to compare
- tied too closely to one agent or tool

This creates a problem.

If the workflow only works with one specific tool or one specific setup, serious users will eventually outgrow it.

Teams do not want to be trapped inside someone else's full harness. They want control.

The Harness Company exists to provide that control.

---

## 3. What Are We Building?

We are building an open-source, local-first harness runtime.

A runtime is the system that coordinates the workflow.

It decides:

- what step runs first
- what data gets passed to the next step
- what happens when a step succeeds
- what happens when a step fails
- what happens when a step rejects the task
- what gets logged
- what artifacts are saved
- how the workflow can be inspected later

The runtime is not the agent itself.

The runtime is the control layer around the agentic workflow.

A simple way to think about it:

```txt
Agents do the work.
The harness runtime controls the workflow.
Plugins customize the workflow.
Artifacts show what happened.
Logs make the workflow inspectable.
```

Another way to think about it:

```txt
workflow file = the program
plugins = the building blocks
runtime = the engine that runs the program
artifacts and logs = the record of what happened
```

---

## 4. What Does “Programming an Agent Workflow” Mean?

It does not mean users need to write a new general-purpose programming language.

It means users can define an agent workflow in a structured file.

That file says:

- run this plugin first
- pass its output to the next plugin
- continue if it succeeds
- stop or ask for help if it rejects
- fail clearly if something errors
- save artifacts so the run can be inspected later

This lets developers design agent behavior without hardcoding everything into one custom script or one fixed product.

---

## 5. Why Not Build One Full Harness?

Selling a complete end-to-end harness sounds attractive, but it has a major weakness.

Every serious developer, team, or company has different needs.

They may want different:

- AI models
- coding agents
- planning systems
- testing tools
- repository structures
- approval processes
- security rules
- deployment systems
- internal workflows

A fixed harness forces users to adopt our way of working.

That is not the right approach.

Instead, The Harness Company should provide the system that lets users build their own way of working.

The better product is not:

> Use our complete workflow.

The better product is:

> Use our runtime to build the workflow that fits your team.

---

## 6. Why Open Source?

The harness architecture should be open source and free.

This matters because the system is infrastructure.

Developers are more likely to trust infrastructure when they can inspect it, run it locally, and understand what it is doing.

Open source helps us:

- build trust
- attract developers
- get feedback early
- encourage plugin creation
- make the runtime easier to adopt
- avoid forcing monetization too early
- let the architecture prove itself through usage

The goal at this stage is adoption and credibility, not immediate monetization.

---

## 7. What About Monetization?

The core runtime should remain free and open source.

The early goal is adoption, trust, and developer usefulness — not immediate marketplace monetization.

The long-term business model can come from a plugin ecosystem built around the runtime.

Possible future revenue streams include:

- first-party premium plugins
- paid third-party plugins
- marketplace transaction fees
- paid plugin verification or certification
- private plugin registries for teams
- enterprise governance and approval tools

The marketplace should not be part of V1.

The correct sequence is:

```txt
Open-source runtime
→ local plugin system
→ useful first-party plugins
→ public plugin registry
→ verified plugins
→ paid plugin marketplace
```

The runtime is the wedge.

First-party plugins are the second wedge.

They prove what the runtime can do before a large third-party ecosystem exists.

The marketplace becomes valuable only after developers already want to install and use plugins.

---

## 8. Who Is This For?

The early audience is technical builders who are already experimenting with AI agents.

This includes:

- solo developers building agent workflows
- AI tooling researchers
- open-source agent builders
- developer tooling teams
- startups experimenting with agentic coding
- teams that want more control over AI-assisted software work

These users are not looking for a shiny black-box product.

They want a system they can inspect, customize, and extend.

---

## 9. What Makes This Useful?

The Harness Company runtime should make agent workflows easier to:

- program
- compose
- run
- test
- inspect
- replay
- debug
- compare
- modify

For example, a developer should be able to write a workflow file, run it, inspect where it failed, replay from a specific step, and swap out one plugin without rebuilding the entire system.

Another important feature is run comparison.

Agent workflows can behave differently across runs. A small change in the model, plugin version, workflow file, repository state, or input context can change the final result.

The runtime should help users compare two runs and see what changed.

For example:

```txt
Run 1 used planner-plugin v1.0 and passed validation.
Run 2 used planner-plugin v1.1 and failed validation.
The diff shows where the workflow diverged.
```

This makes agent behavior easier to debug instead of forcing developers to guess what happened.

That is the practical value.

The early selling point is not a grand claim about becoming a standard.

The early selling point is local usefulness.

If the runtime is useful enough, the artifact formats and workflow patterns may become a standard later through adoption.

---

## 10. Market Positioning

The Harness Company should be positioned as infrastructure for agentic workflows.

It is not competing directly as another AI coding agent.

It is not trying to replace every existing tool.

Instead, it should make existing and future tools easier to compose into reliable workflows.

The positioning should be:

> A local-first, open-source control plane for agentic workflows.
>
> A declarative workflow layer for programming agentic systems with plugins.
>
> A debugging layer for understanding why agentic workflows changed between runs.

This means users should be able to bring their own tools and still use The Harness Company runtime to coordinate the workflow.

---

## 11. How We Penetrate the Market

The early market strategy should be developer-led.

### Step 1 — Open-source the runtime

Make the core runtime free and public.

Developers should be able to inspect it, run it locally, and understand the design.

### Step 2 — Make local debugging excellent

The runtime should help users write, run, inspect, and debug agent workflows locally.

This is the wedge.

Useful commands could include:

```bash
harness workflow validate workflow.yaml
harness workflow explain workflow.yaml
harness run workflow.yaml
harness inspect run_001
harness replay run_001
harness diff run_001 run_002
```

If the local developer experience is strong, adoption becomes much easier.

### Step 3 — Provide clear examples

The first examples should use mock plugins.

The point is to show the architecture, not overwhelm people with real integrations too early.

### Step 4 — Ship useful first-party plugins

The runtime should not depend on community plugins at the beginning.

The first useful plugins should come from The Harness Company.

Good early plugin candidates include:

- intent router
- atomic task planner
- repo knowledge index
- local shell executor
- test validator
- coding-agent adapter

These plugins should prove why the runtime matters.

The goal is not to build every plugin ourselves forever.

The goal is to give developers enough useful examples that they understand why they would build or install more plugins later.

### Step 5 — Add real adapters later

After the runtime is stable, we can add integrations for existing tools.

Examples may include coding agents, validators, test runners, CI systems, or repository context tools.

### Step 6 — Let plugin usage create the ecosystem

The goal is not to declare a standard.

The goal is to create a useful system that people naturally want to build plugins for.

If enough people build around the runtime, the standard emerges from usage.

---

## 12. What We Should Avoid Early

The project should avoid overpromising.

Early versions should not claim:

- perfect security sandboxing
- universal agent compatibility
- enterprise governance
- marketplace readiness
- automatic workflow intelligence
- full monetization clarity
- industry-standard status
- claiming the workflow format is already a full programming language
- adding complex language features too early
- dynamic loops without strong circuit breakers
- arbitrary inline code inside workflow files
- depending on community plugins before first-party plugins prove value
- treating `diff` as a minor utility instead of a core debugging feature
- making the plugin contract vague

Those claims are premature.

The first version should be honest and focused.

It should prove that plugin-based agentic workflows can be programmed, composed, run, inspected, compared, and modified locally.

---

## 13. Why This Has a Real Chance

This idea has a real chance because agentic tooling is still early and fragmented.

Many people are building agents, but fewer are building the stable workflow layer around them.

As agent use grows, users will need better ways to control, test, and debug these workflows.

The Harness Company can become useful by being the layer that makes agentic workflows more understandable and modifiable.

The opportunity is not to own every plugin or every workflow.

The opportunity is to own the runtime layer that helps people compose their own workflows.

---

## 14. The Core Bet

The core bet is this:

> Agentic workflows will become more modular, and developers will need a runtime that lets them program, compose, inspect, compare, and modify those workflows without being locked into one tool.

The Harness Company should build that runtime.

Start open source.

Start local-first.

Start with developer utility.

Sell plugins later only after the runtime earns trust.
