---
name: ui-combobox
description: Read and select Windows ComboBox values with expansion, semantic option matching, and dependent-state reinspection.
---

# ComboBox operations

Read the current selection first. Expand through `ExpandCollapsePattern`, record
the complete live item list in UI order, resolve one option by exact semantic
value, select it, collapse when needed, and read the value again. Never infer
items from documentation or a different module instance.

When an explicitly authorized test pairs the ComboBox with Apply, require one
unique enabled Apply button in the same composite group. Capture any guard that
changes scope (for example, Apply to All Channels) before the first selection.
For each live item: select once, verify the selected value, click Apply once,
record modal title/text/buttons or stable-page response, and verify the value
again. Do not automatically retry selection or Apply after uncertain completion.
Order the original item last so its Apply is also the restoration transaction;
verify the original value and scope guards after completion.

Prefer the deterministic read-only ComboBox inventory script before planning a
multi-item test. It must record the current item, the complete ordered Items
list, same-group Apply identity, and scope-guard state without expanding,
selecting, or clicking. A generated TC remains a candidate until deterministic
validation and explicit mutation authorization both succeed.
