---
name: ui-button
description: Operate Windows UI buttons with Invoke-first execution, state guards, and post-action verification.
---

# Button operations

Use only registered actions permitted by the Capability Model. Resolve the generated component key and require a unique, enabled target. Prefer `InvokePattern`; use an ordinary click only when the driver exposes it as a registered fallback. Never use an unverified coordinate.

Before invocation, classify the button as navigation, reversible, destructive, or modal-triggering from the domain Skill and capability metadata. Define an observable expected outcome, invoke once, re-inspect, and verify that outcome. A disabled button is state evidence, not permission to click by another mechanism.

Treat every state-changing Button as non-idempotent when completion is uncertain.
Never retry its click after a timeout, provider error, missing window, or unknown
modal. Record the input evidence and observed event, then either run only the
predeclared reacquisition/restoration procedure or abort. A batch means a
serial list of independently traced Button steps; it never means concurrent or
unobserved clicks. After a terminal failure, mark all remaining Button steps
`NOT_RUN` so no target is silently omitted or double-triggered.
