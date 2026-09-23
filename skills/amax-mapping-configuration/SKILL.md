---
name: amax-mapping-configuration
description: Plan AMAX-5070 Modbus and EtherCAT mapping tests with channel identity, conflict checks, and restoration.
---

# Mapping configuration tests

Identify mappings by device/module/channel semantics, not row number. Read the relevant row set before change, enforce published address/range constraints, and reject duplicate or overlapping assignments. Apply only explicitly authorized reversible changes, verify the persisted mapping after re-navigation, and restore the original row values.
