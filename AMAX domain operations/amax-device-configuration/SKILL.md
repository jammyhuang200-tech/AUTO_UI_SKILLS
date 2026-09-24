---
name: amax-device-configuration
description: Plan AMAX-5070 information and device-configuration tests with save-dialog evidence and guaranteed restoration.
---

# Device configuration tests

Firmware and device identity fields are observations unless a registered, authorized capability explicitly permits change. For device-description testing, capture the original value, set only the approved test value, invoke the registered Apply action, capture the result dialog, verify persistence after navigation, then restore and apply the original value. Firmware update and Online Update remain blocked from automatic confirmation.
