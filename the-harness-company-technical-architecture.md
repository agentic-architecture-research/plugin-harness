# The Harness Company — Technical Architecture & Build Plan

## 1. Project Definition

**The Harness Company** is building an open-source, local-first harness runtime for programming, testing, inspecting, and modifying agentic workflows through plugins.

The runtime is not an end-to-end agent. It is the control plane that lets users define how agentic workflows are assembled, executed, inspected, and improved.

The core product is:

> A local-first runtime where developers program agentic workflows by composing plugins.

The runtime should make it possible to plug in different agents, tools, planners, validators, context systems, and execution backends later without rewriting the whole workflow architecture.

V1 should focus on the plugin architecture, workflow runner, artifact model, event log, and local developer experience.

---

## 2. Core Thesis

Agentic workflows should not be hardcoded into one monolithic harness.

Different teams will want different:

- agents
- planners
- executors
- validators
- context systems
- review gates
- logging formats
- CI/CD flows
- repository structures

The Harness Company should provide the stable runtime layer that allows those pieces to be composed safely and predictably.

The goal is not to force users into one workflow. The goal is to give them a runtime for building their own workflow.

---

## Workflow-as-Program Model

The workflow file is not just configuration.

In The Harness Company runtime, the workflow file is the user-facing programming model for agentic systems.

A workflow defines:

- which plugin steps run
- what order they run in
- what input each step receives
- where each step sends its output
- what happens on success
- what happens on reject
- what happens on error
- what artifacts are created
- what execution path can be inspected or replayed later

A simple mental model:

```txt
workflow.yaml = program
plugin = callable module/function
plugin manifest = function signature and metadata
inputSchema/outputSchema = type contract
runtime = interpreter
artifact store = program state/output
event log = execution trace
inspect/replay/diff = debugging tools
```

V1 should not become a general-purpose programming language.

V1 should remain a declarative workflow language based on DAGs, typed plugin contracts, explicit routing, artifacts, and logs.

---

## 3. V1 Scope

V1 should be intentionally narrow.

### Included in V1

- plugin contract
- plugin registry
- declarative workflow language / workflow file format
- DAG-based workflow runner
- artifact store
- event log
- schema validation
- advisory permission manifests
- marketplace-compatible plugin metadata
- local CLI
- mock plugins for proof of architecture
- basic inspect/replay commands
- repo structure and examples

### Explicitly Not Included in V1

- true OS-level sandboxing
- plugin marketplace
- cloud dashboard
- monetization system
- automatic cyclic workflow execution
- enterprise governance layer
- remote plugin execution
- private plugin registry
- complex multi-agent orchestration

V1 should prove that the runtime can load plugins, compose a workflow, pass artifacts, validate inputs and outputs, log execution, and allow one plugin implementation to be swapped for another.

---

## 4. Runtime Responsibilities

The runtime is responsible for the following:

1. Load and parse the workflow file.
2. Discover and register plugins.
3. Validate plugin metadata.
4. Validate step input before plugin execution.
5. Run plugins in graph order.
6. Validate plugin output after execution.
7. Store artifacts.
8. Write event logs.
9. Track workflow state.
10. Route success, reject, and error outcomes.
11. Prevent uncontrolled execution loops.
12. Support inspection and replay.
13. Validate workflow language rules.
14. Resolve step dependencies and input references.
15. Produce clear validation errors before execution.

The runtime should not care what the plugin does internally. It only cares about the contract.

---

## 5. Plugin Contract

Every plugin should be treated as a black box with declared behavior.

A plugin must declare:

- manifest metadata
- execution function

Example TypeScript shape:

```ts
export type HarnessPlugin = {
  manifest: PluginManifest;

  run(input: PluginInput, context: RuntimeContext): Promise<PluginResult>;
};

export type PluginManifest = {
  id: string;
  name: string;
  version: string;
  description: string;

  author: {
    name: string;
    url?: string;
  };

  runtimeVersion: string;
  capabilities: string[];

  inputSchema: unknown;
  outputSchema: unknown;

  permissions: PluginPermissions;

  license: {
    type: "open-source" | "source-available" | "commercial";
    name?: string;
  };

  pricing: {
    type: "free" | "paid";
  };

  links?: {
    homepage?: string;
    repository?: string;
    documentation?: string;
  };

  marketplace?: {
    listed: boolean;
    verified: boolean;
    signed: boolean;
  };
};
```

The runtime validates the shape before allowing the plugin to participate in a workflow.

---

## Plugin Runtime Contract

The plugin contract is the most important boundary in the runtime.

A plugin should be easy to replace, test, validate, and inspect without requiring the runtime to understand the plugin's internal logic.

Every plugin receives:

- structured input
- runtime context
- artifact access helpers
- event/logging helpers
- declared permission information
- workflow/run metadata

Every plugin returns a `PluginResult`.

The runtime is responsible for validating the plugin's declared input and output schemas before allowing the workflow to continue.

### PluginInput

`PluginInput` is the validated data passed into a plugin step.

It may come from:

- inline workflow input
- output from a previous step
- an artifact reference
- runtime-provided metadata
- user-approved external input

Example:

```ts
export type PluginInput = {
  value: unknown;
  source: "inline" | "step-output" | "artifact" | "runtime" | "user";
  sourceRef?: string;
};
```

### RuntimeContext

`RuntimeContext` is the controlled interface between the plugin and the runtime.

Plugins should not directly mutate workflow state.

Instead, they should interact with the runtime through explicit context helpers.

Example:

```ts
export type RuntimeContext = {
  runId: string;
  stepId: string;
  workflowName: string;

  artifacts: {
    read(ref: string): Promise<unknown>;
    write(name: string, value: unknown): Promise<ArtifactRef>;
  };

  events: {
    emit(event: RuntimeEvent): Promise<void>;
  };

  permissions: PluginPermissions;

  logger: {
    info(message: string, data?: unknown): void;
    warn(message: string, data?: unknown): void;
    error(message: string, data?: unknown): void;
  };
};
```

### Side Effects

Plugins may need to perform side effects such as reading files, writing files, running commands, or calling network services.

In V1, side effects are not fully sandboxed.

However, plugins must declare intended side effects in their manifest.

The runtime should:

- display declared side effects before execution
- include side effects in logs
- warn on broad permissions
- make side effects visible during inspection
- prepare the system for stricter enforcement later

The runtime should not claim full side-effect prevention in V1.

### Artifact References

Plugins should pass large or durable outputs through artifacts instead of raw in-memory objects when possible.

Example:

```ts
export type ArtifactRef = {
  artifactId: string;
  path: string;
  mediaType: "application/json" | "text/markdown" | "text/plain";
  hash?: string;
};
```

This makes plugin outputs easier to inspect, replay, diff, and debug.

---

## Plugin Manifest Strategy

Every plugin should include marketplace-compatible metadata from day one.

This does not mean the marketplace exists in V1.

It means plugins should already declare enough information to support a future registry or marketplace.

The manifest should separate pricing from licensing.

Pricing answers:

> Does the user need to pay to access this plugin?

Licensing answers:

> What is the user legally allowed to do with this plugin?

Example:

```ts
pricing: {
  type: "free"
}

license: {
  type: "open-source",
  name: "MIT"
}
```

Another example:

```ts
pricing: {
  type: "paid"
}

license: {
  type: "commercial"
}
```

This keeps the system flexible without adding marketplace complexity too early.

---

## 6. Plugin Result Model

Plugins should always return one of three known outcomes:

```ts
export type PluginResult =
  | {
      status: "success";
      output: unknown;
      artifacts?: ArtifactRef[];
      events?: RuntimeEvent[];
    }
  | {
      status: "reject";
      reason: string;
      missing?: string[];
      recommendedNextStep?: string;
      artifacts?: ArtifactRef[];
      events?: RuntimeEvent[];
    }
  | {
      status: "error";
      error: string;
      recoverable: boolean;
      artifacts?: ArtifactRef[];
      events?: RuntimeEvent[];
    };
```

### Success

The plugin completed its job and produced valid output.

### Reject

The plugin intentionally refused to continue because required context, authority, input, or permissions were missing.

Reject is not the same as an error.

Example reject cases:

- missing required context
- unclear task boundary
- missing user approval
- invalid workflow state
- insufficient declared permission

### Error

The plugin failed unexpectedly.

Example error cases:

- runtime exception
- invalid internal state
- failed subprocess
- malformed dependency response

This distinction is important because agentic systems need to know whether something failed accidentally or refused correctly.

Rejected or failed plugins may still emit diagnostic artifacts and events. Those outputs should be persisted so users can inspect why the workflow stopped.

---

## 7. Permission Model

V1 permissions are advisory, not fully enforced.

A plugin can declare permissions like:

```yaml
permissions:
  readFiles:
    - "src/auth/**"
  writeFiles:
    - "src/auth/**"
  commands:
    - "npm test"
  network:
    enabled: false
```

In V1, this is a transparency and review mechanism.

The runtime should:

- record declared permissions
- display permission requirements before workflow execution
- include permissions in logs
- warn when permissions are broad
- use permissions for human review and plugin trust scoring later

The runtime should not claim to fully prevent unauthorized OS-level access in V1.

True enforcement is future work and may require:

- Deno-style permissioned execution
- WASM sandboxing
- Docker containers
- Firecracker microVMs
- restricted subprocess runners
- remote isolated execution

V1 should be honest: schema validation and advisory manifests are included; deep sandboxing is not.

---

## 8. Workflow Language Model

A workflow is a declarative program made of plugin steps.

V1 should support Directed Acyclic Graphs only.

That means:

- sequential steps are allowed
- branching is allowed
- backward cycles are not allowed in V1
- automatic replan/execute/validate loops are not allowed in V1

Example workflow config:

```yaml
name: basic-agentic-workflow
version: 0.1.0

steps:
  - id: intake
    plugin: mock-intake
    input:
      source: user-request
    next:
      success: plan
      reject: stop
      error: stop

  - id: plan
    plugin: mock-planner
    inputFrom: intake.output
    next:
      success: execute
      reject: stop
      error: stop

  - id: execute
    plugin: mock-executor
    inputFrom: plan.output
    next:
      success: validate
      reject: stop
      error: stop

  - id: validate
    plugin: mock-validator
    inputFrom: execute.output
    next:
      success: complete
      reject: stop
      error: stop
```

### V1 Workflow Language Rules

A valid workflow must have:

- unique step IDs
- valid plugin references
- valid `next` targets
- no cycles
- explicit `success`, `reject`, and `error` routes
- valid `input` or `inputFrom`
- no references to missing step outputs
- clear terminal states such as `stop` or `complete`
- plugin inputs that match the plugin input schema
- plugin outputs that match the plugin output schema

The workflow validator should fail fast before execution when possible.

Future versions may compile workflow files into a normalized internal representation before execution.

This would make workflows easier to validate, visualize, replay, diff, and eventually optimize.

---

## 9. Circuit Breakers

Even though V1 should avoid cycles, the runtime should still include basic circuit breaker concepts early.

Minimum controls:

- max workflow steps per run
- max runtime duration
- max plugin execution duration
- max artifact size
- max log size
- clear stop state

Future cyclic workflows must include:

- maxRetries
- timeoutMs
- loop history
- failure fingerprinting
- repeated-output detection
- manual approval gates

This prevents agentic workflows from entering infinite loops and burning compute or API credits.

---

## 10. Artifact Model

The runtime should pass structured artifacts between plugins.

Artifacts should be stored locally in a `.harness/` directory.

Suggested artifacts:

```txt
.harness/
  runs/
    run_001/
      workflow.yaml
      state.json
      events.jsonl
      artifacts/
        intake.output.json
        plan.output.json
        execute.output.json
        validate.output.json
      reports/
        summary.md
```

Artifacts should be:

- local-first
- human-readable where possible
- machine-parseable
- replayable
- diffable
- easy to inspect from CLI

The artifact format should become useful before it is marketed as a standard.

---

## 11. Event Log

Every important runtime action should be written to `events.jsonl`.

Example event:

```json
{
  "timestamp": "2026-04-30T12:00:00Z",
  "runId": "run_001",
  "stepId": "plan",
  "pluginId": "mock-planner",
  "eventType": "plugin.completed",
  "status": "success",
  "artifact": "artifacts/plan.output.json"
}
```

Events should support:

- inspection
- debugging
- replay
- comparison across runs
- future analytics
- future hosted collaboration tools

---

## Run Comparison and Diffing

Run comparison should be treated as a first-class feature.

Agentic systems are non-deterministic. The same workflow may produce different results because of:

- different model outputs
- different plugin versions
- different workflow versions
- different input artifacts
- different repository state
- different context selection
- different validation results

The runtime should make these differences visible.

`harness diff run_001 run_002` should compare:

- workflow file versions
- plugin IDs and versions
- step execution paths
- step status changes
- input artifacts
- output artifacts
- event traces
- validation results
- runtime duration
- error/reject reasons

This is one of the main ways The Harness Company makes agentic workflows debuggable.

The goal is not only to know that two runs were different.

The goal is to understand where and why they diverged.

---

## 12. CLI Requirements

The CLI is critical for adoption.

V1 should include commands like:

```bash
harness init
harness plugin list
harness plugin info <plugin-id>
harness plugin permissions <plugin-id>
harness plugin validate ./plugin
harness plugin pack ./plugin
harness workflow validate workflow.yaml
harness workflow explain workflow.yaml
harness workflow graph workflow.yaml
harness run workflow.yaml
harness inspect run_001
harness replay run_001 --from plan
harness diff run_001 run_002
harness diff run_001 run_002 --artifacts
harness diff run_001 run_002 --events
harness diff run_001 run_002 --workflow
```

Later registry commands should include:

```bash
harness plugin search <query>
harness plugin install <plugin-id>
harness plugin update <plugin-id>
```

Payment commands should not be added yet.

The CLI should make the local runtime obviously useful.

The early adoption wedge is not “adopt our standard.”

The wedge is:

> Use this runtime because it makes agentic workflows easier to run, inspect, replay, debug, and modify.

---

## 13. Suggested Repository Structure

```txt
the-harness-company-runtime/
  README.md
  package.json
  tsconfig.json

  src/
    cli/
      index.ts
      commands/
        init.ts
        run.ts
        inspect.ts
        replay.ts
        diff.ts
        validate.ts
        plugin-list.ts

    runtime/
      workflow-runner.ts
      plugin-registry.ts
      artifact-store.ts
      event-log.ts
      state-store.ts
      graph-validator.ts
      schema-validator.ts
      permission-manifest.ts
      circuit-breaker.ts

    types/
      plugin.ts
      plugin-manifest.ts
      pricing.ts
      license.ts
      marketplace.ts
      workflow.ts
      workflow-language.ts
      workflow-step.ts
      workflow-route.ts
      artifact.ts
      event.ts
      result.ts
      permissions.ts
      runtime-context.ts

    schemas/
      plugin.schema.ts
      plugin-manifest.schema.ts
      workflow.schema.ts
      workflow-language.schema.ts
      artifact.schema.ts
      result.schema.ts

    utils/
      path-utils.ts
      jsonl.ts
      hashing.ts
      timestamps.ts

  examples/
    basic-dag-workflow/
      workflow.yaml
      plugins/
        mock-intake.ts
        mock-planner.ts
        mock-executor.ts
        mock-validator.ts

    branching-workflow/
      workflow.yaml
      plugins/
        mock-router.ts
        mock-task-a.ts
        mock-task-b.ts

  docs/
    architecture.md
    plugin-contract.md
    workflow-language.md
    workflow-examples.md
    artifact-model.md
    event-log.md
    permissions.md
    cli.md
    v1-scope.md

  tests/
    runtime/
      workflow-runner.test.ts
      graph-validator.test.ts
      plugin-registry.test.ts
      artifact-store.test.ts
      event-log.test.ts

    examples/
      basic-dag-workflow.test.ts
```

---

## 14. Recommended Tech Stack

### Runtime Language

TypeScript is the best starting point.

Reasons:

- strong type system
- good CLI ecosystem
- natural fit for JSON/YAML schemas
- familiar to web and agent tooling developers
- easy npm distribution
- compatible with many existing AI toolchains

### Suggested Libraries

- `commander` or `oclif` for CLI
- `zod` for schema validation
- `yaml` for workflow config parsing
- `execa` for controlled subprocess calls later
- `fast-glob` for file pattern matching
- `jsonc-parser` if config comments are desired
- `vitest` for tests

### Package Format

Start with npm package distribution.

Example:

```bash
npm install -g @theharnesscompany/runtime
```

or local project install:

```bash
npm install -D @theharnesscompany/runtime
```

---

## 15. Build Order

### Phase 1 — Skeleton Runtime

- define core types
- define PluginInput
- define RuntimeContext
- define ArtifactRef
- define RuntimeEvent
- create CLI shell
- load workflow YAML
- validate workflow shape
- validate plugin manifest metadata
- support pricing/license metadata as non-enforced fields
- load local mock plugins
- execute linear workflow
- write event log
- write artifacts

### Phase 2 — DAG Runner

- add graph validation
- support branching
- prevent cycles
- improve state tracking
- add stop/complete states

### Phase 3 — Developer Experience

- add `inspect`
- add `replay`
- add first-class run diffing for workflows, plugin versions, artifacts, events, and step outcomes
- add better error output
- add example workflows
- improve documentation

### Phase 4 — Plugin Authoring Kit

- plugin template generator
- plugin manifest generator
- plugin metadata validator
- plugin schema helpers
- plugin testing helper
- example plugin contract tests
- plugin packaging guide
- plugin contract documentation
- side-effect declaration examples
- artifact read/write examples
- reject/error handling examples

### Phase 5 — Real Adapters Later

Only after the core is stable should real integrations be added.

### First-Party Plugin Strategy

Community plugins should not be required for early value.

The Harness Company should ship a few useful first-party plugins after the core runtime works.

Suggested first-party plugins:

- intent-router plugin
- atomic-task-planner plugin
- repo-knowledge-index plugin
- local shell executor plugin
- test-runner validator plugin
- GitHub Actions validator plugin
- Playwright validator plugin
- Codex CLI executor plugin
- Claude Code executor plugin

The first-party plugin goal is to prove that the runtime is useful before a public plugin ecosystem exists.

### Phase 6 — Public Plugin Registry

- plugin publishing flow
- searchable public registry
- install by plugin ID
- plugin metadata pages
- version compatibility checks
- permission display
- free plugin distribution

### Phase 7 — Paid Marketplace

- publisher accounts
- paid plugin listings
- marketplace transaction fees
- plugin purchase flow
- license access checks
- verified plugin badges
- private/team registry support

---

## 16. Testing Strategy

The runtime should be tested before real plugins exist.

Test with fake plugins first.

Core tests:

- valid workflow loads
- invalid workflow fails
- duplicate step IDs are rejected
- missing plugin references are rejected
- invalid next targets are rejected
- invalid inputFrom references are rejected
- unreachable steps are warned or rejected
- plugin input schema is enforced
- plugin output schema is enforced
- plugin receives correct RuntimeContext
- plugin cannot directly mutate workflow state
- plugin can write artifacts through runtime context
- plugin artifact references are persisted correctly
- rejected plugin can still emit artifacts and events
- failed plugin can still emit diagnostic events
- artifacts are written
- events are written
- DAG cycles are rejected
- reject state stops or routes correctly
- error state stops or routes correctly
- replay starts from selected artifact
- workflow explain output matches the resolved graph
- workflow graph output matches the resolved DAG
- diff detects changed plugin version
- diff detects changed workflow version
- diff detects changed artifact output
- diff detects changed execution path
- diff detects success-to-reject and success-to-error changes

This proves the architecture before integrations complicate the project.

---

## 17. Definition of Done for V1

V1 is done when a user can:

1. Install the runtime locally.
2. Initialize a `.harness/` project.
3. Define an agentic workflow program in YAML.
4. Register mock plugins.
5. Run the workflow.
6. Inspect the run.
7. Replay from a step.
8. Diff two runs.
9. Swap one plugin implementation without changing the runtime.
10. Understand all outputs from local artifacts and logs.
11. Validate a workflow before running it.
12. Understand the resolved execution graph before running it.
13. Run `harness diff` to compare two workflow executions.
14. See changed plugin versions, artifacts, events, and step outcomes between runs.
15. Build a simple plugin using the documented PluginInput, RuntimeContext, and PluginResult contracts.

That is enough to prove the architecture.

---

## 18. Strategic Technical Principle

The Harness Company should not start by building the marketplace.

It should start by making the open-source runtime and local plugin system useful.

However, every plugin should be structured as if it may eventually appear in a registry or marketplace.

That means V1 should include strong plugin metadata, version compatibility, permissions, schemas, pricing metadata, and license metadata.

The marketplace comes later.

The runtime earns trust first.
