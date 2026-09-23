---
name: ui-device-tree
description: Navigate device Tree/TreeItem controls by semantic path with expansion and post-navigation reinspection.
---

# Device tree operations

Resolve one root and traverse an explicit semantic path. At each level, expand only when necessary, re-enumerate visible children, and require a unique match before continuing. After selecting a device or module, re-inspect the detail pane and verify device identity. Never substitute similarly named modules or rely on a stale tree item reference after expansion.
