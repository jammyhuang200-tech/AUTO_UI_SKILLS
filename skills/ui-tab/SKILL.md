---
name: ui-tab
description: Navigate Windows tab controls and re-inspect the selected page before resolving child components.
---

# Tab operations

Select tabs by stable page semantics through the registered navigation action. Verify current page after selection and always re-inspect before resolving child targets; cached child elements are invalid after navigation. Tab navigation is not evidence that page content loaded; verify an expected page component.
