---
name: auto-test-plan-authoring
description: Create and offline-validate a product-specific V4 Execution Plan from a User Goal using the current Page Object View, Registry, and Method Catalog in a product_ui_automation project. Use for plan authoring or repair; never execute UI or assume mutation authorization.
---

# Automatic V4 test-plan authoring

Create one product-specific semantic Execution Plan from the User Goal, save it
under the product project, and prove that the current Validator accepts it. Plan
authoring is offline by default and must not create a UI side effect.

## Output boundary

Produce the project's V4 Execution Plan, normally:

```text
execution_plan/<plan-id>.json
```

Do not confuse it with a global UI policy document, a legacy action plan, a
Managed Script package, or physical UI instructions. The Plan may contain only
fields accepted by the current project schema and Validator.

## 1. Locate `product_ui_automation`

Search the current workspace for a directory named `product_ui_automation`.
Accept it as `PROJECT_ROOT` only when its active project structure includes the
relevant equivalents of:

```text
run_v4.py
execution_plan/
page_views/
capabilities/
validation/
ui/method_registry.py
```

Do not invent the path or substitute an unrelated automation project. If more
than one candidate exists, use explicit user or workspace context; otherwise
stop and ask for the intended project. If the project is absent, report that
the installed Skill alone does not contain the product model or runtime.

Before planning, read the project's active architecture, Planner contract,
Page View index, capability documentation, Plan examples, and Validator entry
point. Use project-relative paths rather than assuming one developer's absolute
filesystem layout.

## 2. Read and normalize the User Goal

Extract:

- product page or domain;
- target state or information requested;
- assertions and evidence expected;
- repetition or branching requirements;
- scope limits;
- whether the goal requires a UI mutation.

Do not silently add operations, pages, devices, channels, values, or assertions
that the user did not request. Resolve harmless wording ambiguity from the
current capability model; ask for direction when different interpretations
would materially change the test or mutation scope.

## 3. Classify Read-only versus Mutation

Classify every required operation before retrieving writable methods:

- **Read-only** observes state without changing selection, input, configuration,
  output, device mode, files, firmware, or connectivity.
- **Mutation** includes any selection, toggle, text entry, Apply, Output,
  Download, Reset, Update, topology rescan, mode change, or action that can
  change application or device state.

Navigation is read-only only when the product contract explicitly treats it as
non-mutating.

## 4. Confirm authorization scope

Authorization must be explicit in the current request and must cover the exact
product, target, operation, values, and breadth of change. Do not inherit it
from earlier runs, historical results, example Plans, writable controls, or the
mere existence of mutation methods.

Without explicit mutation authorization:

- do not retrieve an authorized/writable catalog;
- do not add mutation calls to the Plan;
- do not silently replace the requested write test with a different read-only
  test;
- report the missing scope needed to continue.

Plan authoring authorization is not execution authorization. Even when an
authorized mutation Plan is created, this Skill performs offline validation
only.

## 5. Retrieve the current Page Object View

Read the relevant View under `PROJECT_ROOT/page_views/` plus its current:

```text
manifest.json
runtime_bindings.json
validation.json
```

Treat historical values and `last_verified_run` as provenance, not live state.
Reject `pending_v4_binding`, `historical_unregistered`, proposed contracts, and
entries without a current runtime binding.

## 6. Retrieve candidates from Registry and Method Catalog

Use the project's local capability retrieval interface when available. Inspect
`python run_v4.py --help` and the capability documentation before composing the
command. A supported V4.1 project may provide a read-only retrieval form such
as:

```powershell
python run_v4.py --profile <profile> `
  --goal "<normalized-user-goal>" `
  --candidate-limit 5
```

Use target filters or required responsibility coverage when the project
supports them. Candidate retrieval must not execute UI.

Cross-check every candidate against the current Registry and exported Method
Catalog. The Registry is the source of callable target instances; the Method
Catalog is the source of method names, exact argument names, types, returns,
Preconditions, and Observations.

## 7. Accept only runtime-published targets

A call is eligible only when all of the following hold:

- the Page Object View marks it `runtime_published`;
- `runtime_target` exists in the current Registry;
- the method is explicitly published for that component type;
- its argument contract is available;
- its authorization class is compatible with the confirmed scope.

Never promote a historical or proposed method during plan authoring. If no
eligible target exists, return a Capability Gap instead of fabricating a target,
method, locator, or low-level action.

## 8. Select invariant-preserving atomic methods

Choose the smallest published method that owns the full product invariant. For
operations requiring selection plus Apply, modal handling, verification,
cleanup, or restoration, select one atomic component method that owns that
sequence.

Do not decompose an invariant into independently callable unsafe operations.
Never place locator, AutomationId, coordinate, UIA pattern, wait, retry, modal,
cleanup, restoration, driver, or arbitrary Python details in the Plan.

## 9. Bind arguments, results, Preconditions, and Observations

For every call:

- use the exact semantic `target` and published `method`;
- bind only declared argument names and compatible values;
- use `save_as` when later calls, assertions, or analysis require the result;
- use result or loop-variable references only after they are defined;
- add bounded `foreach` or `if` only when the current Plan schema supports it.

Preconditions and Observations are Method Catalog contracts. Use them to choose
the method and determine required result bindings; do not serialize new
`precondition` or `observation` Plan fields unless the current schema explicitly
defines them. Do not expand them into hidden low-level calls.

## 10. Build the Execution Plan

Read the schema version from the active project and follow validated local
examples. A V4-compatible Plan normally has this shape:

```json
{
  "schema_version": "4.0",
  "plan_id": "product-goal-readable-id",
  "calls": [
    {
      "target": "product.page.component",
      "method": "published_method",
      "args": {},
      "save_as": "result_name"
    }
  ]
}
```

Create a concise lowercase hyphenated `plan_id` describing the product goal.
Do not overwrite an existing Plan with different content. If the intended path
already exists, compare it and request direction or choose a user-approved new
ID.

## 11. Save the Plan

Write valid UTF-8 JSON to:

```text
PROJECT_ROOT/execution_plan/<plan-id>.json
```

Keep the file deterministic: stable call order, consistent indentation, no
runtime state, timestamps, secrets, credentials, screenshots, or machine-local
absolute paths.

## 12. Run offline Validator

From `PROJECT_ROOT`, validate with the product's supported Python environment:

```powershell
python run_v4.py `
  --profile <profile> `
  --plan execution_plan/<plan-id>.json
```

Do not add any of these by default:

```text
--execute
--authorized
--pid
```

Their absence is a safety requirement, not an optimization. The validation run
must not attach to, navigate, or modify the application.

## 13. Repair from structured validation errors

When validation fails, use its structured error code and path to make the
narrowest correction:

- `UNKNOWN_TARGET`: re-check runtime bindings and Registry; never rename by
  guesswork.
- `UNKNOWN_METHOD`: select an actually published method; do not expose a Python
  attribute directly.
- argument or type error: use the exact Method Catalog signature.
- undefined result or variable: fix ordering or binding.
- invalid control flow: reduce nesting or batch size to schema limits.

Re-run the same offline command after each correction. Stop when the same
Capability Gap remains, the required mutation scope is not authorized, or a
valid correction would change the User Goal. Do not weaken the Validator,
schema, Registry, or Method Catalog to make a Plan pass.

## 14. Never execute UI without separate authorization

Completion of this Skill means the Plan exists and offline validation passed.
It does not authorize or perform UI execution, Managed Script publication,
device mutation, or evidence cleanup.

Report:

```text
project root
plan path and plan_id
profile
Read-only or Mutation classification
authorization scope used for authoring
selected runtime targets and methods
Validator status
confirmation that no UI execution occurred
```

If the user later requests execution, treat that as a separate operation and
reconfirm all live-execution prerequisites and authorization at that time.

## Completion criteria

Finish only when:

- the Plan represents the User Goal without added scope;
- every target and method is currently runtime-published;
- atomic methods preserve required invariants;
- arguments and result bindings match the current catalog;
- the file is saved under `execution_plan/` without overwriting unrelated work;
- the offline Validator passes;
- no `--execute`, `--authorized`, or `--pid` was implicitly added;
- no UI side effect occurred.
