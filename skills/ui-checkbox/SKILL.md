---
name: ui-checkbox
description: Read and set Windows CheckBox state idempotently with TogglePattern and state verification.
---

# Checkbox operations

Read `ToggleState` before changing it. Compare the observed state with the requested state and do nothing when they already match. Otherwise use a registered TogglePattern-based action, then read state again and require `On` or `Off` as expected. Treat `Indeterminate` explicitly; never blindly toggle it.

Fallback order is registered TogglePattern action, registered invoke/click, then keyboard Space. An observation-bound coordinate is the final locator fallback and still requires confidence at or above policy threshold.
