---
name: ui-edit
description: Read and safely update Windows Edit controls using ValuePattern, read-only checks, verification, and restoration.
---

# Edit operations

Read with `ValuePattern` when available. Before mutation, require a unique enabled component, `writable=true`, an allowed registered action, and explicit mutation authority. Capture the original value before the first change. Set the value through the registered `fill` action, read it back, and verify exact semantics required by the test.

For reversible tests, register cleanup before mutation and restore the captured value even after a later failure. Do not infer writability from visual appearance or an invented AutomationId.
