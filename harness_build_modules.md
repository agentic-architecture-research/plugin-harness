# Harness Runtime Build Modules

## Purpose

Define the high-level build modules for The Harness Company runtime.

The goal is to build the runtime in dependency order:

```txt
plugin contracts -> workflow composition -> validation -> execution -> inspection -> ecosystem readiness
```

---

## 1. Plugin Boundary + Contract Module

### Purpose

Define what a plugin is and what the runtime is allowed to know about it.

The runtime should treat plugins as black boxes. It validates and executes the boundary, not the plugin internals.

### Includes

- `PluginManifest`
- plugin identity: `id + version`
- plugin metadata
- `PluginInput`
- `RuntimeContext`
- `PluginResult`
- input schema
- output schema
- advisory permissions
- license metadata
- pricing metadata

### Core Output

```ts
run(input: PluginInput, context: RuntimeContext): Promise<PluginResult>
```

### Why It Exists

Everything else depends on a stable plugin boundary.

Without this module, the runtime cannot safely load, validate, inspect, replace, or compose plugins.

---

## 2. Workflow Language / Workflow IR Module

### Purpose

Define how users compose plugins into a workflow.

V1 workflow syntax can be YAML, but the runtime should normalize workflows into an internal representation before execution.

### Includes

- workflow file format
- step model
- plugin references
- input references
- route table
- terminal states
- `WorkflowIR`
- future Harness Language compile target

### Core Shape

```ts
type WorkflowIR = {
  name: string;
  version: string;
  steps: IRStep[];
  terminals: TerminalState[];
};
```

### Why It Exists

The workflow is the user-facing programming model.

The future Harness Language should compile into the same IR, not require a separate runtime.

---

## 3. Validation Module

### Purpose

Reject invalid plugins, workflows, schemas, routes, and graphs before execution.

### Includes

- plugin manifest validation
- plugin result shape validation
- workflow shape validation
- step ID validation
- route validation
- plugin reference validation
- input reference validation
- schema validation
- schema compatibility validation
- graph validation
- permission manifest validation
- runtime compatibility checks

### Validation Layers

```txt
syntax validation
semantic validation
schema validation
graph validation
policy validation
```

### Why It Exists

The runtime should fail early with clear errors.

Agentic workflows are already non-deterministic; the control plane should not add avoidable ambiguity.

---

## 4. Plugin Registry + Resolution Module

### Purpose

Discover, register, and resolve plugins by exact identity.

### Includes

- local plugin discovery
- plugin registry
- `id + version` lookup
- runtime compatibility lookup
- manifest access
- input/output schema access
- permissions access
- capability lookup

### Required Operations

```ts
resolvePlugin(id: string, version: string): PluginManifest;
getInputSchema(pluginRef): Schema;
getOutputSchema(pluginRef): Schema;
getPermissions(pluginRef): PluginPermissions;
checkRuntimeCompatibility(pluginRef, runtimeVersion): boolean;
```

### Why It Exists

A workflow cannot execute vague plugin names.

Exact plugin resolution is required for reproducibility, replay, diffing, validation, and future registry support.

---

## 5. Runtime Execution Engine Module

### Purpose

Execute validated workflows in graph order.

### Includes

- DAG runner
- step scheduler
- step execution
- state store
- route handling
- success/reject/error control flow
- stop/complete states
- runtime-owned workflow state
- `RuntimeContext` injection
- circuit breakers

### Control Flow

```txt
success -> next step
reject  -> configured reject route
error   -> configured error route
```

### Why It Exists

This is the control plane.

It coordinates plugins without becoming the plugin, agent, planner, executor, or validator itself.

---

## 6. Artifact + Event Log Module

### Purpose

Persist what happened during each run.

### Includes

- artifact store
- artifact references
- run directory structure
- `events.jsonl`
- structured runtime events
- step outputs
- diagnostic artifacts
- state snapshots
- run metadata

### Suggested Structure

```txt
.harness/
  runs/
    run_001/
      workflow.yaml
      state.json
      events.jsonl
      artifacts/
        step.output.json
      reports/
        summary.md
```

### Why It Exists

Agentic workflows must be inspectable, replayable, diffable, and debuggable.

Artifacts are durable values. Events are the execution trace.

---

## 7. Developer CLI + Debugging Module

### Purpose

Expose the runtime through a local developer interface.

### Includes

- `harness init`
- `harness plugin list`
- `harness plugin info`
- `harness plugin validate`
- `harness workflow validate`
- `harness workflow explain`
- `harness workflow graph`
- `harness run`
- `harness inspect`
- `harness replay`
- `harness diff`

### Why It Exists

The CLI is the adoption wedge.

Developers should be able to run, inspect, replay, debug, compare, and modify workflows locally.

---

## 8. Plugin Authoring + Ecosystem Readiness Module

### Purpose

Make it easy to build valid plugins and prepare for a future plugin ecosystem.

### Includes

- plugin template generator
- manifest generator
- schema helpers
- plugin testing helpers
- plugin contract tests
- packaging guide
- example plugins
- permission examples
- artifact examples
- reject/error examples
- marketplace-compatible metadata

### Why It Exists

The runtime should not depend on community plugins early, but it should be easy for plugins to exist later.

V1 should remain local-first and open source while keeping plugin metadata ready for a future registry or marketplace.

---

# Recommended Build Order

```txt
1. Plugin Boundary + Contract
2. Workflow Language / Workflow IR
3. Validation
4. Plugin Registry + Resolution
5. Runtime Execution Engine
6. Artifact + Event Log
7. Developer CLI + Debugging
8. Plugin Authoring + Ecosystem Readiness
```

---

# Dependency Logic

```txt
Plugin Boundary
  -> defines valid plugin units

Workflow IR
  -> defines valid plugin composition

Validation
  -> proves composition is safe before execution

Plugin Registry
  -> resolves exact plugin implementations

Runtime Engine
  -> executes the validated graph

Artifacts + Events
  -> make execution inspectable and replayable

CLI
  -> exposes the runtime to developers

Authoring Kit
  -> helps others build compatible plugins
```

---

# V1 Success Criteria

V1 is successful when a developer can:

- define a workflow
- register mock plugins
- validate the workflow
- run the workflow
- inspect the run
- replay from a step
- diff two runs
- swap a plugin implementation
- write a simple plugin against the documented contract

That proves the runtime architecture before real integrations, registries, or marketplace features are added.

