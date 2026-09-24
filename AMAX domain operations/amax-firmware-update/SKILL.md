---
name: amax-firmware-update
description: Inspect AMAX-5070 firmware-update UI while blocking automatic update confirmation and connectivity-impacting execution.
---

# Firmware update safety

Treat firmware update as `prohibited-automatic`. The Agent may read installed version, inspect selector state, and, when the requirement asks, open a confirmation dialog without confirming it. It must not select firmware, start transfer, confirm update, restart, or reinterpret general mutation authority as firmware-update authority. Record the blocked operation and required human authorization.
