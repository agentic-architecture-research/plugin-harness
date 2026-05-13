# Plugin Boundary

## Simple Definition

A **plugin boundary** is the contract between the Harness runtime and a plugin.

It defines what the runtime must know in order to safely load, validate, run, inspect, and replace a plugin without needing to understand the plugin's internal logic.

The plugin can do the work however it wants.

The runtime only controls the boundary.

---

## Core Principle

The plugin system should be opinionated about **contracts**, not behavior.

```txt
Runtime controls: connection, validation, execution boundary, result shape, artifacts, logs, permissions.
Plugin controls: internal logic, tools used, model used, API calls, algorithms, strategy.
```

A plugin may use:

- an LLM
- a local model
- a shell command
- an API
- a rule engine
- a validator
- a planner
- a human approval step
- any internal workflow

The runtime should not care, as long as the plugin obeys the contract.

---

## What the Runtime Needs to Know

The runtime needs enough information to answer these questions:

```txt
Can I identify this plugin?
Can I validate its manifest?
Can I validate its input?
Can I run it?
Can I validate its output?
Can I understand its result?
Can I inspect what it was allowed to do?
Can I log what happened?
Can I swap it with another plugin safely?
```

If the runtime can answer those questions, the plugin boundary is strong enough for V1.

---

## Minimal Plugin Shape

A plugin should expose two things:

```ts
export const manifest = {
  id: "example-plugin",
  name: "Example Plugin",
  version: "0.1.0",
  runtimeVersion: "0.1.0",
  capabilities: ["validation"],
  inputSchema: {},
  outputSchema: {},
  permissions: {},
  license: {
    type: "open-source",
    name: "MIT"
  },
  pricing: {
    type: "free"
  }
};

export async function run(input, context) {
  return {
    status: "success",
    output: {}
  };
}
```

That is the basic integration point.

---

## Required Manifest Fields

A plugin manifest should declare:

| Field | Purpose |
|---|---|
| `id` | Stable plugin identifier |
| `name` | Human-readable plugin name |
| `version` | Plugin version |
| `description` | What the plugin does |
| `author` | Plugin author or maintainer |
| `runtimeVersion` | Compatible Harness runtime version |
| `capabilities` | What kind of work the plugin performs |
| `inputSchema` | Shape of input the plugin accepts |
| `outputSchema` | Shape of output the plugin produces |
| `permissions` | Intended side effects or access requirements |
| `license` | Legal usage model |
| `pricing` | Free or paid access model |

The manifest makes the plugin discoverable, inspectable, and replaceable.

---

## Required Runtime Function

Every plugin should provide a `run()` function.

```ts
run(input: PluginInput, context: RuntimeContext): Promise<PluginResult>
```

The runtime passes input and context into the plugin.

The plugin returns a result.

The plugin should not directly mutate runtime state.

---

## Plugin Input

Plugin input is the validated data passed into the plugin.

It may come from:

- inline workflow input
- output from a previous step
- an artifact reference
- runtime metadata
- user-approved external input

Example:

```ts
export type PluginInput = {
  value: unknown;
  source: "inline" | "step-output" | "artifact" | "runtime" | "user";
  sourceRef?: string;
};
```

The runtime should validate input before the plugin runs.

---

## Runtime Context

Runtime context is the controlled interface between the plugin and the runtime.

It gives the plugin access to runtime services without letting it directly control workflow state.

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

The context should provide controlled access to:

- run metadata
- step metadata
- artifact helpers
- event helpers
- logger helpers
- declared permissions

---

## Plugin Result

Every plugin should return one of three statuses:

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

### `success`

The plugin completed its job and produced valid output.

### `reject`

The plugin intentionally refused to continue because something required was missing.

Examples:

- missing context
- missing permission
- unclear task boundary
- missing user approval
- invalid workflow state

Reject is not a crash. It is a controlled refusal.

### `error`

The plugin failed unexpectedly.

Examples:

- runtime exception
- failed subprocess
- malformed dependency response
- invalid internal state

Error means something broke.

---

## Permissions

Permissions describe intended side effects.

Example:

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

In V1, permissions should be treated as **advisory**, not full sandbox enforcement.

The runtime should:

- display declared permissions
- log declared permissions
- warn on broad permissions
- make side effects visible during inspection
- prepare for stricter enforcement later

The runtime should not claim it can fully prevent unauthorized OS-level access in V1.

---

## What the Runtime Should Enforce

The runtime should enforce:

```txt
Manifest exists.
Manifest has required fields.
Input schema exists.
Output schema exists.
Plugin result uses only success, reject, or error.
Reject result includes a reason.
Error result includes error message and recoverable flag.
Plugin output matches declared output schema.
Plugin does not mutate runtime state directly.
Artifacts and events go through runtime context.
```

These rules make plugins loadable, testable, inspectable, and replaceable.

---

## What the Runtime Should Not Enforce Internally

The runtime should not enforce how a plugin performs its work.

Avoid rules like:

```txt
Plugin must use an LLM.
Plugin must use one model provider.
Plugin must follow one agent architecture.
Plugin must be a planner, executor, or validator.
Plugin must store internal state in one specific way.
Plugin must follow The Harness Company workflow philosophy internally.
```

Those rules would make the system too opinionated.

The plugin boundary should allow many types of plugins to integrate.

---

## Good Boundary vs Bad Boundary

### Good Boundary

```txt
Here is the manifest shape.
Here is the input schema.
Here is the output schema.
Here are the allowed result statuses.
Here are the declared permissions.
Here is the runtime context.
Return a valid result.
```

### Bad Boundary

```txt
Use this exact agent pattern.
Use this exact model.
Use this exact planner.
Use this exact internal workflow.
Use this exact storage strategy.
Think about tasks exactly the way we do.
```

The first approach creates interoperability.

The second approach creates lock-in.

---

## Why This Matters

The Harness runtime should be a control plane, not a monolith.

A strong plugin boundary means:

- plugins can be replaced
- workflows can be inspected
- outputs can be validated
- errors can be understood
- side effects can be reviewed
- future registries can trust metadata
- users can bring their own tools
- The Harness Company does not need to predict every use case

The runtime owns the contract.

The plugin owns the work.

That is the clean separation.

---

## One-Line Summary

A plugin boundary defines how a plugin connects to the runtime, what it must declare, what it may receive, what it must return, and how the runtime can validate and inspect it without controlling the plugin's internal behavior.
