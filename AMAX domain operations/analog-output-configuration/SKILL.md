---
name: analog-output-configuration
description: Operate and verify AMAX analog-output pages containing channel output, range, slew-rate, safety, and under-4mA settings.
---

# Analog Output Configuration

## Scope

Use this Skill for the AMAX-5024 `AO` page. Treat the output-value panel,
Selected Items groups, and Channel Information as one composite domain surface.

## Read-only inventory

1. Select the module by exact TreeItem name and re-inspect.
2. Verify `Module Information` and `AO` TabItems.
3. Read the selected channel, output value, range, slew rate, safety value, and
   under-4mA value only when the provider exposes them.
4. Read checkbox states without toggling them.
5. For Channel Information expansion, use `$column-treeview` and execute its
   verified focus and keyboard sequence. Record the visual
   expansion separately from provider-exposed descendants.
6. Record `columnTreeViewChInfo` as provider-opaque when UIA exposes only its
   ScrollBar and omits owner-drawn rows or cells.
7. Return to Module Information.

## Mutation policy

Without explicit authorization, never invoke `Output`, `Set Safety`, or any
`Apply` button; never move `tBarOutputVal`; never alter a ComboBox, Edit, or
CheckBox. `Apply to All Channels` increases the scope and requires separate,
explicit all-channel authorization.

For an authorized future write, capture channel identity and every dependent
original state first, apply only the requested setting, verify the device
readback, and restore and re-verify the original state.

For an explicitly authorized ComboBox suite, inventory the complete live Item
list before planning and pair each ComboBox with one unique enabled Apply in its
own composite group. Keep Apply to All Channels Off. Preserve related enable
states without toggling them. Within each case, exercise every non-original
Item once, apply the original Item last, and verify restoration before starting
the next ComboBox. Record modal or stable-AO response and post-Apply readback
for every Item. Never retry selection or Apply after uncertain completion.

Observed on AMAX-5024(S2), Utility 2.7.04 B06: every live SlewRate Item
produced a stable AO page without a modal, while every live Range Item produced
one `Information` modal that was captured and confirmed. Post-Apply readback
matched every selected Item. The all-channel guard remained Off, SlewRate
Enable remained On, and both original selections were applied last and
verified restored. For maintained Item lists and response signatures, read
[`references/observed-combobox-behavior.json`](references/observed-combobox-behavior.json)
when analyzing a new execution. Matching behavior needs no Skill change;
differences produce a review candidate rather than an automatic edit.

## Fallback

Prefer AutomationId and UIA patterns, then semantic keyboard navigation.
Coordinate input is last resort and must never be guessed when geometry is not
available.
