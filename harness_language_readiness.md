# Harness Language Readiness

## Purpose

Define the runtime constraints needed today so a future **Harness Language** can compile into the current plugin workflow runtime without architectural rewrites.

The language should be a higher-level syntax for composing plugin boundaries, not a language for implementing plugin internals.

---

## Core Model

```txt
Harness Language Source
  -> parser
  -> AST
  -> semantic validator
  -> normalized workflow IR
  -> runtime execution plan
  -> plugin runtime
```

The future language should compile down to the same execution model used by V1 workflows.

```txt
plugin = callable primitive
manifest = callable signature
inputSchema = input type contract
outputSchema = output type contract
PluginResult = control-flow contract
permissions = side-effect contract
artifacts = durable value references
event log = execution trace
runtime = interpreter/executor
```

---

## Non-Goal

The Harness Language should not define plugin internals.

It should not specify:

- model provider
- prompting strategy
- agent architecture
- internal state format
- planning algorithm
- execution strategy
- storage implementation

Plugins remain black boxes behind declared contracts.

---

## Required Runtime Invariants

The runtime must preserve these invariants now:

```txt
1. Every plugin has a stable id + version.
2. Every plugin exposes a manifest.
3. Every plugin declares inputSchema and outputSchema.
4. Every plugin returns a PluginResult.
5. Every step has explicit success/reject/error routing.
6. Every artifact has a stable reference.
7. Every execution emits structured events.
8. Workflow state is runtime-owned.
9. Plugins access runtime services only through RuntimeContext.
10. Workflow graphs are validated before execution.
```

These invariants become the target assumptions for the future compiler and validator.

---

## Plugin as Language Primitive

A plugin call should eventually map to a language-level operation.

Example conceptual form:

```txt
let plan = call @harness/planner@1.0.0(input)
let result = call @harness/executor@1.0.0(plan.output)
call @harness/validator@1.0.0(result.output)
```

Compiled form:

```yaml
steps:
  - id: plan
    plugin: "@harness/planner"
    version: "1.0.0"
    input: ...
    next:
      success: execute
      reject: stop
      error: stop
```

The language call is valid only if the plugin boundary is valid.

---

## Compile Target: Workflow IR

Before runtime execution, the language should compile into a normalized internal representation.

Suggested IR shape:

```ts
type WorkflowIR = {
  name: string;
  version: string;
  steps: IRStep[];
  terminals: TerminalState[];
};

type IRStep = {
  id: string;
  pluginRef: {
    id: string;
    version: string;
  };
  input: InputExpression;
  routes: {
    success: RouteTarget;
    reject: RouteTarget;
    error: RouteTarget;
  };
};
```

The runtime should execute IR, not source syntax directly.

This keeps YAML workflows, future `.harness` source files, generated workflows, and visual editors on the same backend.

---

## Validation Layers

Validation should be layered.

```txt
Syntax validation
  - source parses correctly
  - valid declarations
  - valid call expressions

Semantic validation
  - plugin references resolve
  - plugin versions resolve
  - step IDs are unique
  - route targets exist
  - terminal states exist

Type/boundary validation
  - input expression matches plugin inputSchema
  - producer outputSchema is compatible with consumer inputSchema
  - PluginResult branches are handled
  - artifact references are valid

Graph validation
  - no cycles in V1
  - no missing dependencies
  - no invalid inputFrom references
  - no unreachable required steps

Policy validation
  - permissions are declared
  - broad permissions warn
  - unsupported permissions fail or warn depending on runtime mode
```

---

## Boundary Checks Needed for Language Support

The runtime/plugin registry should expose enough metadata for the future language checker.

Required lookup operations:

```ts
resolvePlugin(id: string, version: string): PluginManifest
getInputSchema(pluginRef): Schema
getOutputSchema(pluginRef): Schema
getPermissions(pluginRef): PluginPermissions
getCapabilities(pluginRef): string[]
checkRuntimeCompatibility(pluginRef, runtimeVersion): boolean
```

Required validation operations:

```ts
validateInput(pluginRef, value): ValidationResult
validateOutput(pluginRef, value): ValidationResult
validateResultShape(result): ValidationResult
validatePermissionManifest(pluginRef): ValidationResult
validateRouteTable(routes): ValidationResult
```

---

## Result as Control Flow

`PluginResult` should be treated as a control-flow primitive.

```ts
type PluginResult =
  | { status: "success"; output: unknown; artifacts?: ArtifactRef[] }
  | { status: "reject"; reason: string; missing?: string[]; artifacts?: ArtifactRef[] }
  | { status: "error"; error: string; recoverable: boolean; artifacts?: ArtifactRef[] };
```

Language-level routing should compile to explicit route tables.

```txt
success -> next step
reject  -> stop | repair | request-input | diagnostics
error   -> stop | retry | diagnostics
```

V1 should require explicit handling for all three statuses.

---

## Schema Compatibility

The language validator needs schema compatibility checks.

Minimum rule:

```txt
producer.outputSchema must be assignable to consumer.inputSchema
```

Example:

```txt
planner.outputSchema -> executor.inputSchema
executor.outputSchema -> validator.inputSchema
```

If compatibility cannot be proven statically, the compiler should either:

```txt
- reject the program
- require an explicit adapter plugin
- require an explicit cast/unsafe boundary in future versions
```

V1 should prefer rejection or adapter plugins over unsafe casts.

---

## Artifact References as Values

Artifacts should be representable as typed references.

```ts
type ArtifactRef = {
  artifactId: string;
  path: string;
  mediaType: string;
  hash?: string;
};
```

Future language expressions may reference artifacts:

```txt
artifact("plan.output.json")
step("plan").artifact("output")
```

Runtime requirement:

```txt
artifact references must be stable, serializable, inspectable, and replayable
```

---

## Permissions as Policy Surface

Permissions should remain part of the manifest even before enforcement is complete.

Future language uses:

```txt
- static permission warnings
- policy gates
- approval requirements
- sandbox selection
- marketplace trust metadata
- CI validation
```

Example policy rule:

```txt
A workflow cannot call a plugin with writeFiles unless the workflow declares file-write approval.
```

V1 requirement:

```txt
permissions are declared, logged, displayed, and available to validators
```

---

## Versioning Requirement

Plugin identity must be versioned.

```txt
pluginRef = id + version
```

Example:

```yaml
plugin: "@harness/planner"
version: "1.0.0"
```

The future language must not compile against ambiguous plugin names.

Changing plugin versions should affect:

```txt
- validation
- execution trace
- replay
- diff
- reproducibility checks
```

---

## Event Log as Execution Trace

The event log should be sufficient to reconstruct language-level execution.

Required event fields:

```txt
runId
workflowId
stepId
pluginId
pluginVersion
inputRef
outputRef
status
routeTaken
artifactRefs
errorReason
rejectReason
timestamp
```

Future debugger features depend on this trace.

---

## Current Build Priorities

To stay language-ready, build these first:

```txt
1. PluginManifest schema
2. PluginResult schema
3. RuntimeContext contract
4. Plugin registry with id + version resolution
5. Workflow IR type
6. Workflow validator
7. Schema compatibility validator
8. Graph validator
9. Artifact store
10. Event log
11. Inspect/replay/diff support
```

Avoid adding syntax-level language features until these contracts are stable.

---

## Design Rule

Do not build a language runtime separate from the workflow runtime.

Build one stable execution backend:

```txt
YAML workflow -> WorkflowIR -> Runtime
Harness language -> WorkflowIR -> Runtime
Visual editor -> WorkflowIR -> Runtime
Generated workflow -> WorkflowIR -> Runtime
```

The language should be syntax over the same validated plugin composition model.

---

## Final Principle

```txt
The Harness Language should program plugin composition, not plugin implementation.
```

The runtime should be built now around stable plugin boundaries, typed inputs/outputs, explicit result routing, artifacts, permissions, and event traces so the language can compile into it later.

