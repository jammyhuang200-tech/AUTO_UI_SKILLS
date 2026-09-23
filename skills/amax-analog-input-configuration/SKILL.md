---
name: amax-analog-input-configuration
description: Plan readback and explicitly authorized configuration tests for AMAX analog-input module pages such as AMAX-5017V and AMAX-5017C AI.
---

# Analog input configuration tests

Inventory Module Information before opening AI. On AI, treat Sampling Rate, Burnout Detect mode and enablement, Apply to All Channels, and Range as a single channel-configuration domain: changing any selector or checkbox is a mutation even before Apply is pressed.

Without explicit write authorization, read current values and states only. Do not invoke Locate Enable, Download, Reset Module, configuration Load/Save, Trend Log, or any Apply button. Record controls whose values or channel rows are not exposed by the legacy provider as `provider-omitted`; do not infer them from labels or prior scans.

For a future authorized write test, capture the selected channel, all affected selector and checkbox states, and whether Apply to All Channels is active before mutation. Apply through the unique button in the same GroupBox, verify the intended channel scope, then restore and reapply every original state. Never treat authorization for one AI setting as authorization for firmware, reset, Locate, file, or all-channel changes.

For AMAX-5017V Range tests, `cbxRange` and the same-group Apply form one
transaction. Inventory the live Range list before planning. When Apply to All
Channels is Off, exercise only the current channel; do not toggle the guard.
Apply every item once and keep the original Range as the final item so the last
Apply restores device state. Record, per item, selection verification, one Apply
input, Information modal or stable AI response, post-Apply value, and duration.
An unexpected modal, changed item list, changed all-channel guard, or uncertain
Apply completion captures evidence and aborts without an automatic retry; a
separate cleanup Apply may restore only the captured original Range.

Observed on AMAX-5017V(S0), Utility 2.7.04 B06: `cbxRange` exposed, in UI
order, `+/-150 mV`, `+/-500 mV`, `+/-1 V`, `+/-5 V`, `+/-10 V`,
`0~150 mV`, `0~ 500 mV`, `0~1 V`, `0~5 V`, and `0~10 V`. Applying each
item while Apply to All Channels was Off produced no modal and kept the AI page
stable; post-Apply readback matched every selected item. Allow a bounded
no-modal observation period before classifying this response as `stable-ai`.
The tested per-item durations were approximately 14.27–14.89 seconds. The
original `+/-5 V` item was applied last and both the Range and all-channel guard
were verified restored.

When analyzing a new Range execution, compare its deterministic signature with
[`references/observed-behavior.json`](references/observed-behavior.json). If it
matches, generate the normal report without proposing a Skill change. If it
differs, create an evidence-backed Skill update candidate for human review;
never edit this Skill automatically from a single run.
