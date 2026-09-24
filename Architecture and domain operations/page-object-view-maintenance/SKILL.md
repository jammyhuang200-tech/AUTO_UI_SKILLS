---
name: page-object-view-maintenance
description: Maintain, regenerate, and audit V4.1 Page Object Views and their source-to-runtime bindings in test-plans/festo_ui_automation. Use when UI scans, control repositories, Typed Components, published methods, generated catalogs, Page Views, or UI maps change; do not use for ordinary test execution.
---

# V4.1 Page Object View maintenance

Maintain the offline capability model consumed by the Agent Planner. A Page
Object View is defined before planning; the Planner reads it but never creates
or promotes capabilities during a test run.

## Locate the project

Find the repository root containing:

```text
test-plans/festo_ui_automation/
```

Treat that directory as `PROJECT_ROOT`. In the primary Windows workspace it is:

```text
C:\Users\jammy\Desktop\Codex\test-plans\festo_ui_automation
```

Before changing generation rules or contracts, read completely:

```text
PROJECT_ROOT/README.md
PROJECT_ROOT/references/agent_ui_automation_formal_architecture.md
PROJECT_ROOT/page_views/README.md
PROJECT_ROOT/planner/agent_planner_contract.md
```

## Architectural boundary

Keep these responsibilities separate:

```text
Runtime scan and maintained sources
    -> generated locator/catalog metadata
    -> Typed Components and published methods
    -> Page Object View and runtime bindings
    -> Agent Planner
    -> Execution Plan
    -> Validator and Runner
```

- Page Object Views are offline capability projections, not Execution Plans.
- Historical observations are evidence, not current UI state.
- A View never grants mutation authority.
- Only `runtime_published` methods with a valid `runtime_target` may be planned.
- `pending_v4_binding` is design work and exposes no executable method.
- `historical_unregistered` must not invent a locator or runtime target.
- Precondition and Observation are semantic contracts, not extra UI steps.
- Locator, wait, modal, retry, cleanup, and restoration remain inside Typed
  Components and the execution framework.

If no published method preserves the required invariant, report a capability
gap. Do not bypass it with UIA calls, coordinates, arbitrary Python, or a loose
sequence of lower-level methods.

## Page Object View lifecycle

1. Capture or receive a verified source scan. A live scan is a separate UI
   operation and requires the authorization appropriate to that operation.
2. Compare it with the maintained source. Preserve unknown and provider-omitted
   values rather than inferring them.
3. Update the appropriate source under `controls/`, `fixtures/`, or another
   declared source path. Record provenance for imported live data.
4. Regenerate locator/catalog metadata. Never patch generated output to hide a
   stale source.
5. Implement or update the Typed Component and its explicit published method
   map when a new runtime capability is required.
6. Regenerate Page Object Views and runtime bindings.
7. Run generator checks and offline regression tests.
8. Review the diff from source through generated output before allowing the
   Agent Planner to consume it.

## Project directory contract

Use the following directory names consistently:

| Path | Definition and ownership |
| --- | --- |
| `planner/` | Agent planning contract. It is not an executable Planner implementation. |
| `page_views/` | Generated page-organized capability views, provenance indexes, runtime bindings, and validation output. Do not edit generated YAML directly. |
| `capabilities/` | Published Method Catalog, metadata, and local retrieval used to limit Planner context. |
| `execution_plan/` | V4 schema 4.0 Plans, examples, and generated batches. Plans contain semantic target/method calls, never locator mechanics. |
| `schema/` | JSON schemas for accepted plan or test-plan documents. |
| `validation/` | Structural and Method Catalog validation before execution. |
| `runner/` | Task-agnostic batch invocation and result/report construction. |
| `dispatch/` | Explicit public-method dispatch. Never replace it with unrestricted `getattr`. |
| `ui/method_registry.py` | Stable semantic target-to-component registration. |
| `ui/method_components.py` | Published component implementations and AMAX bindings. |
| `ui/components/` | Shared Typed Component and locator-source models. |
| `ui/generated/` | Generated or generator-backed Typed Component projections. Do not change during normal test execution. |
| `generated/` | Generated Python catalogues such as `page_objects_catalog.py`. |
| `locator/` | Locator resolution, confidence, wait, and recovery mechanics hidden from Plans. |
| `driver/` | UI technology adapters and Desktop Bridge integration. |
| `scanner/` | Scan comparison and programmatic regeneration entrypoints. |
| `controls/` | Maintained control repositories and `manifest.json`; these are code-generation inputs. |
| `source_scan.json` | Baseline multi-tab scan consumed by the main Page Object catalogue generator. |
| `ui_map/` | Generated component-to-skill/risk metadata for UI operations. |
| `maintenance/` | Reviewed scan-diff and locator-promotion decisions; not runtime authority. |
| `fixtures/` | Stable, version-controlled input evidence used by offline generators and tests. |
| `scripts/` | Focused import, scan, generation, and V4 convenience commands. |
| `analyzer/` | Agent result-analysis contract. It does not alter completed execution results. |
| `managed_scripts/` | Python implementation for converting and publishing validated Plans. |
| `managed-scripts/` | Versioned immutable published packages and `repository.json`. The hyphenated directory is data, not the Python package. |
| `artifact_store/` | Runtime artifact creation, indexing, selection, and retention implementation. |
| `artifacts/runs/` | Git-ignored live evidence grouped by Script ID and Run ID. |
| `tests/` | Offline regression tests for contracts, generation, validation, execution, and reports. |

Do not confuse similarly named locations:

```text
managed_scripts/   Python publisher implementation
managed-scripts/   published package repository

generated/         generated Python catalogue
ui/generated/      Typed projections that consume generated metadata

controls/          maintained generator inputs
page_views/        generated Planner-facing capability views
```

## Code-generation procedures

Run commands from `PROJECT_ROOT` with the supported Python environment.

### 1. Baseline Page Object catalogue

Inputs:

```text
source_scan.json
controls/manifest.json
controls/<page>.json
```

Generate:

```powershell
python generate_repository.py
```

Output:

```text
generated/page_objects_catalog.py
```

The generator validates tab counts, control counts, source indexes, and
repository coverage before writing. `ui/generated/controls.py` converts this
catalogue into Typed Component instances.

Verify that committed output is current without rewriting it:

```powershell
python generate_repository.py --check
```

The programmatic maintenance entrypoint is:

```python
from scanner.generator import regenerate
regenerate(check=False)  # generate
regenerate(check=True)   # verify only
```

The current generator intentionally enforces its declared object-count
contract. If the supported scan scope changes, update the source, manifest,
generator expectation, tests, and documentation together; do not merely weaken
the assertion.

### 2. Page Object Views and runtime bindings

Inputs include maintained control repositories, declared fixture evidence, and
the current published component registry used by
`scripts/generate_page_views.py`.

Generate:

```powershell
python scripts/generate_page_views.py
```

Outputs:

```text
page_views/<page>.yaml
page_views/manifest.json
page_views/source_index.json
page_views/runtime_bindings.json
page_views/validation.json
```

The generator must validate:

- source coverage;
- unique page targets;
- published methods only;
- valid Precondition and Observation references;
- invariant-preserving APIs, including no loose `select` publication for an
  `AppliedComboBox`.

Modify a source, registry binding, or generation rule and regenerate. Never
repair these outputs by editing the generated YAML or JSON alone.

### 3. UI operation map

Inputs:

```text
controls/*.json
```

Generate:

```powershell
python ui_map/export_ui_map.py
```

Output:

```text
ui_map/ui_map.json
```

This export assigns semantic tags and recommended UI/domain Skills from control
type and page ownership. Its component-count assertion is a deliberate current
contract; change it only with the complete source and regression update.

### 4. Access Control scan import

Convert a verified read-only Access Control runtime scan into a maintained
control source:

```powershell
python scripts/import_access_control_scan.py <verified-scan.json> `
  --output controls/access_control.json
```

The importer requires the Access Control page to be selected, requires
`mutation_performed: false`, and rejects missing or duplicate AutomationIds.

When a separately authorized setup has produced verified read-only MAC dynamic
field evidence, merge only the expected dynamic controls:

```powershell
python scripts/import_access_control_mac_dynamic.py <mac-scan.json> `
  --controls controls/access_control.json
```

Do not treat the setup authorization or historical scan as authorization for a
later write test.

## Verification

After regeneration:

```powershell
python generate_repository.py --check
python scripts/generate_page_views.py
python ui_map/export_ui_map.py
python -m unittest discover -s tests -v
```

Then review:

```text
page_views/validation.json                 status must be PASS
page_views/source_index.json               every source record must resolve
page_views/runtime_bindings.json           targets and methods must match Registry
page_views/manifest.json                   counts and source hashes must be coherent
generated/page_objects_catalog.py          must match generator output
git diff                                   must show explainable source-to-output changes
```

Do not claim a live UI test from offline generation. Report separately whether
validation was offline, whether a live scan occurred, and whether any mutation
was explicitly authorized.

## Completion criteria

Finish only when:

- the maintained source and provenance explain every generated change;
- generated artifacts are current and reproducible;
- View status accurately distinguishes published, pending, and historical
  entries;
- runtime targets and methods match the Registry and Method Catalog;
- no View or historical value is presented as current live state;
- offline tests pass, or any failure is reported without hiding it through a
  generated-file edit.
