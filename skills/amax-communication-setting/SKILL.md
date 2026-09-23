---
name: amax-communication-setting
description: Plan AMAX-5070 communication-setting tests while protecting connectivity and restoring reversible local settings.
---

# Communication setting tests

Read current mode and dependent field state before planning. A mode selection can change IP-field visibility, writability, and Apply enablement, so re-inspect after every state change. Network mode/address changes, topology rescan, restart, and cable-redundancy changes require separate explicit connectivity-impact authority; otherwise block them before Runner dispatch. Read-only diagnostics remain permitted.

For an explicitly authorized Device Topology Rescan, resolve only the generated `btn_reset_connection` Apply button (`btnResetConnection`) under `gbxResetConnection` / Device Topology Rescan. Do not confuse it with `btn_refresh_topology_status`, which is the Refresh button under Topology Status and does not start a device rescan. Activate the unique Utility window, reacquire Apply by its generated selector, verify that its current bounding rectangle belongs to the Utility process, and send one visible center click. Require a new Loading dialog or progress control to appear, wait for it to close automatically, then require the target device TreeItem and Information page to return. Never confirm another modal, silently retry the click, or treat general mutation authority as rescan authority.

## Authorized Setting Button batch

When the user explicitly authorizes the complete Setting Button batch, define
and validate every target before the first click. Execute serially with the
registered `invoke-setting-button` Action, except Device Topology Rescan which
uses `trigger-device-topology-rescan`. Each target is clicked exactly once.

| Component | Operation | Expected impact | Success and recovery |
|---|---|---|---|
| `btn_apply_host_idle_time` | Reapply current Host Idle Timeout | No disconnect expected | Capture timeout, observe/confirm one declared modal or stable Setting, reacquire and require the value unchanged |
| `btn_tcp_fsv_setting` | Open TCP FSV settings without editing | Opens `Fail Safe Value Setting` secondary window | Require exactly one titled window, inventory its DataGrid/Group ID/Check Topology/Update Topology controls, close it without editing, reacquire Setting, and require FSV state unchanged |
| `btn_apply_fsv` | Reapply current FSV values | Communication FSV reapply | Capture FSV state, observe/confirm modal or stable Setting, require unchanged state |
| `btn_refresh_cable_redundancy_status` | Refresh status | Read refresh | Capture before/after status when exposed and require Setting to remain available |
| `btn_refresh_topology_status` | Refresh topology status | Read refresh; not a rescan | Capture before/after counts when exposed and require Setting to remain available |
| `btn_cable_redund_apply` | Reapply the currently selected cable mode | EtherCAT link may reconfigure | Capture both radio states, confirm declared modal, reacquire the same device and require the original selection |
| `btn_apply_network` | Reapply current network mode/address | Device may temporarily disconnect or rebind | Capture mode and current IP; observe for 30 seconds without retrying; require the unique `Information` modal containing `Change network done!`; confirm `確定`; wait for Information; reacquire the same IP TreeItem, return to Setting, and require original state |
| `btn_reset_connection` | Device Topology Rescan | Loading and page transition | Use the dedicated rescan procedure and require Information after Loading closes |
| `btn_system_reset` | Restart the device | Temporary disconnect/restart | Execute last, confirm the declared modal, wait up to the plan timeout, reacquire the same IP TreeItem and Setting |

Historical evidence shows `btnApplyHostIdleTime` can block a UIA Invoke while
a legacy WinForms modal is active. Therefore use a fresh UIA locator followed
by one verified visible-center click and observe the modal separately. Do not
interpret an Invoke HRESULT or timeout as proof that no click occurred.

Observed Network Apply behavior on `172.16.12.126`: the success dialog can be
delayed beyond eight seconds and temporarily makes the main UIA tree disabled.
After `Change network done!` is confirmed, the page returns to Information and
the accessibility provider can remain empty for several seconds. Treat this as
a recoverable connection transition: poll for the same IP TreeItem, then return
to Setting. Never click Network Apply again merely because reacquisition failed.
Because a legacy `exists` query can itself block beyond the observation deadline,
sample the process-scoped Win32 dialog once more immediately after that query
returns and before recovery begins. If the declared success text is present,
confirm it once; this late sample is observation/recovery, not an action retry.

For every Button record pre-state, single-click input evidence, modal/loading
details, connectivity loss, reacquisition, post-state, duration, success
condition, and recovery method. Unknown modal buttons, ambiguous targets,
unavailable required pre-state, uncertain click completion, failed
reacquisition, or changed configuration invariants are terminal unexpected
events: capture evidence, do not click again, abort the batch, and emit the
remaining steps as `NOT_RUN`. Replanning or extending this Skill occurs only
after the current validated run has ended.

Runtime maintenance evidence from `2026-08-26` proves that
`txtHostIdleTimeout` and `txtFsvTimeout` are unique live AutomationIds even
though the older scan exposed them only as `ID:` names. Promote them through
Runtime Scan → Diff → CodeGen before relying on them as required pre-state.
The observed values (`720` and `99` in that run) are evidence, not constants;
always read them again immediately before a Button operation.

The `2026-08-26` live run also proved that `btnTcpFsvSetting` opens a separate
top-level `Fail Safe Value Setting` window (`TcpWatchDogFsvView`), not a
MessageBox. Its successful Button test is window appearance and inventory. Do
not search for or confirm an OK button. Close the unmodified secondary window,
wait until it disappears, then reacquire the main Utility, target TreeItem, and
Setting page.

The secondary window can appear after the generic three-second Button settling
window because the legacy provider blocks while constructing its DataGrid.
Use the registered 30-second `btnTcpFsvSetting` observation window; do not enter
main-window reacquisition until that dedicated window has either been captured
and closed or timed out.
