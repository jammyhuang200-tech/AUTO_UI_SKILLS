---
name: ui-data-grid
description: Read or edit Windows DataGrid/Table rows by semantic row key and column identity with commit and verification.
---

# DataGrid operations

Resolve the grid, identify exactly one row by a stable semantic key, then locate the target column by header identity. For reads, return the row key, column, and observed cell value as evidence. For edits, first prove the cell is editable and capture its original value; enter edit mode, set the value through a registered action, commit, re-resolve the row, and verify the committed value. Restore through the same row/column identity.

Do not target cells by row index alone because sorting, filtering, and virtualized rows can change positions.
