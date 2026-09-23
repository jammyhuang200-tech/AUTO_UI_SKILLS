# AUTO_UI_SKILLS

Portable personal Codex Skills for UI automation. Built-in `.system` Skills
and plugin caches are intentionally excluded.

## Available Skills

### Agent and device orchestration

- [`amax-5070-desktop-bridge`](skills/amax-5070-desktop-bridge)

### Basic UI components

- [`ui-button`](skills/ui-button)
- [`ui-checkbox`](skills/ui-checkbox)
- [`ui-combobox`](skills/ui-combobox)
- [`ui-edit`](skills/ui-edit)
- [`ui-radiobutton`](skills/ui-radiobutton)
- [`ui-tab`](skills/ui-tab)

### Complex UI components

- [`column-treeview`](skills/column-treeview)
- [`ui-data-grid`](skills/ui-data-grid)
- [`ui-device-tree`](skills/ui-device-tree)
- [`ui-mapping-table`](skills/ui-mapping-table)

## Install in Codex

To install one Skill, enter this in a Codex conversation and replace the final
path segment with the desired Skill name:

```text
$skill-installer 請從以下 GitHub 路徑安裝 amax-5070-desktop-bridge：
https://github.com/jammyhuang200-tech/AUTO_UI_SKILLS/tree/main/skills/amax-5070-desktop-bridge
```

To install all UI component and domain Skills together, enter:

```text
$skill-installer 請從 jammyhuang200-tech/AUTO_UI_SKILLS 安裝以下路徑：
skills/ui-button
skills/ui-checkbox
skills/ui-combobox
skills/ui-edit
skills/ui-radiobutton
skills/ui-tab
skills/column-treeview
skills/ui-data-grid
skills/ui-device-tree
skills/ui-mapping-table
```

The installer places the Skill in `$CODEX_HOME/skills`; when `CODEX_HOME` is
not set, Codex uses `~/.codex/skills`.

The repository layout is compatible with the installer:

```text
AUTO_UI_SKILLS/
  skills/
    amax-5070-desktop-bridge/
      SKILL.md
      agents/
        openai.yaml
    ui-button/
      SKILL.md
    ...
```

## Use after installation

Start a new Codex turn and invoke the Skill explicitly:

```text
$amax-5070-desktop-bridge 請幫我執行 AMAX-5070 UI 自動化任務。
$ui-button 請安全操作這個 Windows 按鈕並驗證結果。
```

It can also trigger automatically when a request matches the `description`
in `SKILL.md`.

## Security

Do not commit `.env` files, API keys, authorization tokens, private keys,
certificates, credentials, or local test evidence containing secrets.
