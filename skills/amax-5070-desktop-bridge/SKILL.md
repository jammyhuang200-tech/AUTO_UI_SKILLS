---
name: amax-5070-desktop-bridge
description: Operate and test the Advantech I/O Module Utility for AMAX-5070 devices through Desktop Bridge and Computer Use. Use for Ethernet discovery, AMAX-5070 TreeItem selection, child-module navigation, TabControl inventory, precise value/state readback, or explicitly authorized writable-field tests.
---

# Desktop UI Automation Skill — AMAX-5070

Operate `Advantech I/O Module Utility (Win32) Version 2.7.04 B06` on the interactive Windows desktop. Use Desktop Bridge for stable selectors and precise values; use Computer Use accessibility text for complete visible hierarchy and legacy controls omitted by Bridge.

Use the project framework at:

```text
C:\Users\jammy\Desktop\Codex\test-plans\festo_ui_automation
```

Read `references/agent_ui_automation_formal_architecture.md` when changing the framework, adding Actions or Assertions, regenerating controls, or analyzing healing. Do not treat that reference as authorization for live mutations.

## Mandatory execution flow

For every test:

1. Read the requested TC YAML completely.
2. Validate it against `schema/test_plan.schema.json` before creating any UI side effect.
3. Create `ExecutionContext`.
4. Execute steps through the stable `Runner`.
5. Use registered Actions and Assertions only.
6. Resolve UI targets through generated Typed Components.
7. Never invent an AutomationId. A raw AutomationId in a TC must uniquely map back to the generated component catalogue.
8. Run step assertions after the Action and top-level assertions after all steps.
9. Save the JSON result and available screenshot, UI tree, log, healing, and maintenance artifacts.

Do not bypass the Runner for ordinary TC execution. Direct Bridge or Computer Use operations are allowed for framework maintenance, runtime scanning, diagnostics, or an explicitly requested ad-hoc UI operation; record that execution mode separately from a Runner TC result.

## Locator policy

Resolve in this order:

1. AutomationId
2. Name + ControlType
3. ClassName + Parent or inspected ancestor path
4. Semantic matching
5. Vision
6. A verified coordinate from the current observation

Never jump directly to Vision or Coordinate. Require exactly one target within the intended parent. Coordinates are last-resort, observation-bound evidence; never reuse a stale screenshot ID, element index, rectangle, or guessed pixel.

## Failure and healing policy

When a locator fails, apply Smart Wait, bounded retry, the next generated locator, self-healing, semantic matching, then vision fallback. Retry at the Step or Locator level; do not blindly rerun the whole TC.

If candidate confidence is below `0.70`, stop without clicking. A fallback success produces `PASS_WITH_HEALING` and records:

- original locator;
- healed locator;
- confidence;
- fallback history;
- screenshot when available;
- UI tree when available.

Healing must not silently or permanently modify generated controls. A failure or healing may produce a maintenance suggestion; applying it is a separate maintenance workflow.

## Modification policy

During normal TC execution, do not modify Runner Core, Driver Core, or `ui/generated/` controls. Do not add conditional business logic or AutomationIds to Runner. Framework maintenance follows:

```text
Runtime Scan → scan.json → Diff → Change Report → CodeGen
             → Generated Typed Components → Regression
```

Page Objects retain business semantics and do not duplicate locators. Application-specific capabilities belong in registered Action plugins. YAML remains declarative: do not add `goto`, `eval`, arbitrary Python, lambdas, scripts, or general try/catch behavior.

## Mandatory test plan

Before every UI operation, read this file completely and follow its rules:

```text
C:\Users\jammy\Desktop\Codex\test-plans\amax-5070-io-module-utility.yaml
```

Record its path and load time in every test result. Prefer `AutomationId`, require exactly one match in the intended parent, and re-inspect after every navigation or action. Do not reuse stale Computer Use element indexes.

Inventory runs are read-only. Do not fill a TextBox or invoke Apply, Setting, Download, Update, Reset, Output, or other state-changing controls unless the user explicitly requests a live write test. For an authorized write test, capture the original value, record and confirm the MessageBox, and restore and verify the original value as required by the plan.

## Attach to the interactive desktop

The required executable is:

```text
C:\Users\jammy\Desktop\Codex\desktop-bridge.exe
```

Prefer a fresh `input` bridge thread:

```powershell
cd C:\Users\jammy\Desktop\Codex
.\desktop-bridge.exe input context
.\desktop-bridge.exe input find --id treeView1
```

Require `windowStation=WinSta0`, `threadDesktop=Default`, and `inputDesktop=Default`. If the bridge cannot attach, start `desktop-bridge.exe serve` beside the Utility in an interactive PowerShell and use `remote`. Do not accept a server running on a Codex sandbox desktop as equivalent to the Utility desktop.

## Discover the target

In the Utility, select the unique `Ethernet` TreeItem, invoke `Tools > Search Device`, and wait until two normalized tree observations are equal. The configured target must resolve exactly once:

```powershell
.\desktop-bridge.exe input find --name "172.16.13.60-[AMAX-5070]" --type TreeItem
.\desktop-bridge.exe input select --name "172.16.13.60-[AMAX-5070]" --type TreeItem
```

After selection, re-inspect even when the legacy provider returns an HRESULT. Trust observed state, not the HRESULT alone.

The expected expanded child TreeItems are:

```text
AMAX-5070 Coupler(G1)
AMAX-5017V(S0)
AMAX-5017C(S1)
AMAX-5024(S2)
```

Find each child by exact Name plus `TreeItem`, require one match under the AMAX-5070 parent, select it, and immediately re-inspect the right-side content.

## Inventory the right-side TabControl

The main AMAX-5070 page uses the outer `tabControlMain` beneath `panelActive > FormAmax5070`. Expanded child modules use `tabControlMain` beneath `panelActive > FormAmax5070IOModule`.

Use fresh Computer Use accessibility text to print the complete visible structure. Do not recursively enumerate this legacy WinForms provider with Bridge `tree`; sibling enumeration can block. Use Bridge `find --id` to supplement exact `value`, `selected`, `toggleState`, `className`, `patterns`, and rectangle data when a stable AutomationId exists.

Expected parent-device tabs:

```text
Information
Setting
Modbus Mapping
Alarm
EtherCAT Mapping
Communication Diagnostics
```

Expected child-module tabs:

```text
AMAX-5070 Coupler(G1): Module Information, Power
AMAX-5017V(S0): Module Information, AI
AMAX-5017C(S1): Module Information, AI
AMAX-5024(S2): Module Information, AO
```

When semantic TabItem selection is omitted by Bridge, focus the verified outer Tab control using keyboard traversal, press Left or Right once, and refresh accessibility state immediately. Never use guessed coordinates.

## Precise state readback

Query stable IDs individually. Examples for AMAX-5070 Setting include:

```powershell
$settingIds = @(
  'txtHostIdleTimeout', 'txtFsvTimeout', 'chbxEnableFsv',
  'txbRing1SlaveCnt', 'txbRing0SlaveCnt', 'txbTopologyScanBusy',
  'txbTotalSlaveCnt', 'txbCableRedundStatus', 'txbCableRedundLinkStatus',
  'rdoCableRedundDisable', 'rdoCableRedundEnable',
  'rdobtStatic', 'rdobtDhcp', 'txbAmax5070SwitchId',
  'txtDefaultGateway', 'txtIPAddress', 'txtSubnetAddress', 'txtMacAddress'
)
foreach ($settingId in $settingIds) {
  .\desktop-bridge.exe input find --id $settingId
}
```

Examples for the AMAX-5024 AO page include:

```powershell
$aoIds = @(
  'txtSelChannel', 'txtOutputVal', 'txtOutUnder4mA', 'txtSafetyValue',
  'cbxSlewRate', 'chbSlewRate', 'chbxEnableSafety',
  'chbOutUnder4mAEnable', 'cbxRange'
)
foreach ($aoId in $aoIds) {
  .\desktop-bridge.exe input find --id $aoId
}
```

Interpret `value` as the editable or combo-box value, `selected` as a radio-button state, and `toggleState` as a checkbox state. Report `null` or `not_exposed` when the provider does not publish a value; never substitute an earlier scan value as live data.

## Test results

For every visited TreeItem and tab, inventory all visible descendants, including disabled controls. Report each control as an object with its Name, AutomationId, ControlType, class, enabled/writable flags, readable value/state, patterns, rectangle, ancestor path, and raw evidence.

For the expanded-child inventory, keep both:

- a flattened `components` list;
- `components_by_page` grouped by `tree_item + tab`.

Preserve raw tool errors and distinguish confirmed, ambiguous, missing, provider-omitted, and blocked states. A complete report does not imply every legacy control exposed every property.

## Troubleshooting

- `No matching UIA element`: refresh the Utility state, verify the device search completed, retry the exact lookup once, then use Computer Use hierarchy as fallback.
- Screenshot error `0x80004002`: use accessibility text and semantic keyboard navigation; do not guess pixels.
- Bridge sees TreeView but omits the right subtree: use Computer Use for hierarchy and Bridge only for stable individual IDs.
- Provider HRESULT after selection: re-inspect before classifying success or failure.
- Missing executable: rebuild `desktop-bridge.exe` with MSVC and the Windows SDK.
