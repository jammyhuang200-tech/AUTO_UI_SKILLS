---
name: ui-radiobutton
description: Inspect and select Windows RadioButton options without accidentally clearing or changing an already-correct group.
---

# RadioButton operations

Read the selected state of the target and its named group. If the requested option is already selected, do nothing. Otherwise require authorization for the resulting setting change, select through `SelectionItemPattern` or a registered fallback, re-inspect the whole group, and verify exactly one intended option is selected. Restoration must select the originally observed option, not toggle the current option.
