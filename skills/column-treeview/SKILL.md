---
name: column-treeview
description: Operate and verify owner-drawn WinForms ColumnTreeView panes whose rows, cells, or context-menu items are omitted by UIA.
---

# ColumnTreeView

Use this Skill for `columnTreeViewChInfo` and equivalent owner-drawn panes. The
control may expose only its Pane and ScrollBar through UIA even when rows and
cells are visibly rendered.

## Expand all

For the verified AMAX-5024 AO Channel Information pane:

1. Reacquire the unique application window and verify the AO page is active.
2. Resolve exactly one `columnTreeViewChInfo` by AutomationId beneath Channel
   Information.
3. Give that pane keyboard focus and capture its control bitmap with pywinauto.
4. Scan every scroll viewport for the rendered tree glyph. A `+` means the
   root is collapsed; a `-` means it is expanded.
5. For each visible `+`, double-click its current row label using coordinates
   relative to the freshly captured control bitmap, then capture again. Never
   reuse a screen coordinate after a row expands or the viewport scrolls.
6. Scroll downward and repeat until the bitmap is unchanged at the bottom and
   no scanned viewport contains a collapsed `+`.

The older `Menu → Down ×2 → Enter` path may be used only when its popup is
visibly present and the selected `Expand All` item can be confirmed. Sending
those keys alone is not success evidence. In the AMAX-5024 AO pane, screenshot-
guided row-label double-click is the proven fallback because UIA omits Row/Cell
nodes and the popup is not reliably exposed.

## Success and evidence

Record all five outcomes:

- the AMAX-5024 AO page remains stable;
- `columnTreeViewChInfo` remains present;
- no unexpected MessageBox appears;
- UIA may expose only ScrollBar descendants and no owner-drawn Row/Cell;
- all child rows are visibly expanded.

The last outcome is visual evidence. Do not claim that UIA proved expansion
when its provider omits the rows. Classify a completed run as
`PASS_WITH_PROVIDER_LIMITATION` when the visual expansion is confirmed and the
first four checks pass.

## Screenshot-only fallback

When UIA omits Row and Cell descendants, resolve exactly one pane by its proven
AutomationId and capture that control with pywinauto `capture_as_image()`.
Prefer the control wrapper; capture the application HWND only when the control
wrapper cannot render. This capture path is independent of other windows
covering the Utility.

The default mode is artifact-only. Save the PNG and record its path, Step ID,
capture method, dimensions, and success/failure status. Do not run OCR, send the
image to an AI visual analyzer, transcribe Cell text, or create structured Cell
values unless a later user request explicitly asks for content extraction.

Bind every image to its plan Step ID:

```text
step_<two-digit-step>_<semantic-label>.png
```

For a changing ComboBox suite, capture after selection, Apply, modal handling,
AO reacquisition, and value verification. Use one distinct file per Item, such
as `step_12_slew_rate_item_01.png`; never overwrite an earlier Item image.

If the control scrolls, save each captured viewport with a distinct ordinal.
The presence of screenshots proves only that image artifacts were saved; do
not claim that all Cells were read or that their content was verified.

## Unexpected events

- Missing or non-unique target: stop before sending input.
- Focus loss or user input between captures: reacquire and restart the visual
  scan from the top; do not reuse the prior hit-test location.
- Unknown popup or MessageBox: capture it and abort without confirmation.
- Computer Use screenshot error `0x80004002`: use the pywinauto HWND/control
  capture fallback, then retain accessibility and keyboard trace evidence;
  never guess coordinates.
- Live polling resets a scrolled viewport: retain the Step-bound screenshot,
  stop after two unchanged ScrollBar Thumb positions, and hand the next fresh
  screenshot to the Agent. The Agent may use the current page-down hit area,
  capture immediately, and continue; never report all cells from a viewport
  that was reset to the top.
- AO or target disappears after a row double-click: record failure and
  reacquire for diagnostics only; do not repeat that click automatically.
