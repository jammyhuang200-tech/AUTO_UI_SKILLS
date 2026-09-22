# AUTO_UI_SKILLS

Portable personal Codex Skills for UI automation. Built-in `.system` Skills
and plugin caches are intentionally excluded.

## Available Skills

### `amax-5070-desktop-bridge`

Operate and test the Advantech I/O Module Utility for AMAX-5070 devices
through Desktop Bridge and Computer Use.

## Install in Codex

In a Codex conversation, enter:

```text
$skill-installer 請從以下 GitHub 路徑安裝 amax-5070-desktop-bridge：
https://github.com/jammyhuang200-tech/AUTO_UI_SKILLS/tree/main/skills/amax-5070-desktop-bridge
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
```

## Use after installation

Start a new Codex turn and invoke the Skill explicitly:

```text
$amax-5070-desktop-bridge 請幫我執行 AMAX-5070 UI 自動化任務。
```

It can also trigger automatically when a request matches the `description`
in `SKILL.md`.

## Security

Do not commit `.env` files, API keys, authorization tokens, private keys,
certificates, credentials, or local test evidence containing secrets.
