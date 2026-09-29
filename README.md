# AUTO_UI_SKILLS

Portable personal Codex Skills for UI automation. Built-in `.system` Skills
and plugin caches are intentionally excluded.

## Available Skills

### Architecture and domain operations

- [`auto-test-plan-authoring`](Architecture%20and%20domain%20operations/auto-test-plan-authoring)
- [`page-object-view-maintenance`](Architecture%20and%20domain%20operations/page-object-view-maintenance)

### Agent and device orchestration

- [`amax-5070-desktop-bridge`](Agent%20and%20device%20orchestration/amax-5070-desktop-bridge)

### Basic UI components

- [`ui-button`](Basic%20UI%20components/ui-button)
- [`ui-checkbox`](Basic%20UI%20components/ui-checkbox)
- [`ui-combobox`](Basic%20UI%20components/ui-combobox)
- [`ui-edit`](Basic%20UI%20components/ui-edit)
- [`ui-radiobutton`](Basic%20UI%20components/ui-radiobutton)
- [`ui-tab`](Basic%20UI%20components/ui-tab)

### Complex UI components

- [`column-treeview`](Complex%20UI%20components/column-treeview)
- [`ui-data-grid`](Complex%20UI%20components/ui-data-grid)
- [`ui-device-tree`](Complex%20UI%20components/ui-device-tree)
- [`ui-mapping-table`](Complex%20UI%20components/ui-mapping-table)

### AMAX domain operations

- [`amax-analog-input-configuration`](AMAX%20domain%20operations/amax-analog-input-configuration)
- [`analog-output-configuration`](AMAX%20domain%20operations/analog-output-configuration)
- [`amax-communication-setting`](AMAX%20domain%20operations/amax-communication-setting)
- [`amax-device-configuration`](AMAX%20domain%20operations/amax-device-configuration)
- [`amax-firmware-update`](AMAX%20domain%20operations/amax-firmware-update)
- [`amax-mapping-configuration`](AMAX%20domain%20operations/amax-mapping-configuration)

## Install in Codex

To install one Skill, enter this in a Codex conversation and replace the final
path segment with the desired Skill name:

```text
$skill-installer 請從以下 GitHub 路徑安裝 amax-5070-desktop-bridge：
https://github.com/jammyhuang200-tech/AUTO_UI_SKILLS/tree/main/Agent%20and%20device%20orchestration/amax-5070-desktop-bridge
```

Architecture Skills use their category path:

```text
$skill-installer 請從以下 GitHub 路徑安裝 auto-test-plan-authoring：
https://github.com/jammyhuang200-tech/AUTO_UI_SKILLS/tree/main/Architecture%20and%20domain%20operations/auto-test-plan-authoring

$skill-installer 請從以下 GitHub 路徑安裝 page-object-view-maintenance：
https://github.com/jammyhuang200-tech/AUTO_UI_SKILLS/tree/main/Architecture%20and%20domain%20operations/page-object-view-maintenance
```

To install all UI component and domain Skills together, enter:

```text
$skill-installer 請從 jammyhuang200-tech/AUTO_UI_SKILLS 安裝以下路徑：
Basic UI components/ui-button
Basic UI components/ui-checkbox
Basic UI components/ui-combobox
Basic UI components/ui-edit
Basic UI components/ui-radiobutton
Basic UI components/ui-tab
Complex UI components/column-treeview
Complex UI components/ui-data-grid
Complex UI components/ui-device-tree
Complex UI components/ui-mapping-table
AMAX domain operations/amax-analog-input-configuration
AMAX domain operations/analog-output-configuration
AMAX domain operations/amax-communication-setting
AMAX domain operations/amax-device-configuration
AMAX domain operations/amax-firmware-update
AMAX domain operations/amax-mapping-configuration
```

The installer places the Skill in `$CODEX_HOME/skills`; when `CODEX_HOME` is
not set, Codex uses `~/.codex/skills`.

The repository layout is compatible with the installer:

```text
AUTO_UI_SKILLS/
  Architecture and domain operations/
    auto-test-plan-authoring/
      SKILL.md
    page-object-view-maintenance/
      SKILL.md
  Agent and device orchestration/
    amax-5070-desktop-bridge/
      SKILL.md
      agents/
        openai.yaml
  Basic UI components/
    ui-button/
      SKILL.md
  Complex UI components/
    column-treeview/
      SKILL.md
  AMAX domain operations/
    amax-analog-input-configuration/
      SKILL.md
```

## Use after installation

Start a new Codex turn and invoke the Skill explicitly:

```text
$auto-test-plan-authoring 請依照這個 User Goal 建立並離線驗證專屬 Execution Plan。
$amax-5070-desktop-bridge 請幫我執行 AMAX-5070 UI 自動化任務。
$ui-button 請安全操作這個 Windows 按鈕並驗證結果。
```

It can also trigger automatically when a request matches the `description`
in `SKILL.md`.

## Security

Do not commit `.env` files, API keys, authorization tokens, private keys,
certificates, credentials, or local test evidence containing secrets.
