---
myst:
  html_meta:
    description: How to report a bug in Ubuntu Core and find further troubleshooting help.
---

(how-to-guides-manage-ubuntu-core-report-a-bug)=
# Report a bug and get help

If you have scanned a QR code from an Ubuntu Core failure screen and reached this page, the device hit an error for which a specific troubleshooting article does not yet exist. The information below points to the best places to get help.

## What to capture before asking for help

If the device booted far enough to give you a shell, the most useful artefacts are:

```bash
journalctl -b
journalctl -b -1
snap version
```

If the device is stuck on the failure screen, photograph the screen itself — the error text and the URL printed alongside the QR code are usually enough to start a diagnosis.

## Where to get help

- **Snapcraft forum** — [forum.snapcraft.io](https://forum.snapcraft.io/c/snapd/5) is the primary place for Ubuntu Core questions. A short forum post that includes the error text, the URL from the QR code, and your device model is usually answered within a day.
- **GitHub issues** — file a bug at [github.com/snapcore/snapd/issues](https://github.com/snapcore/snapd/issues) if you can reproduce the problem and have a journal excerpt to share.
- **Launchpad** — for Ubuntu-archive-related concerns, file at [bugs.launchpad.net/snapd](https://bugs.launchpad.net/snapd).

## See also

- {ref}`Troubleshooting <how-to-guides-manage-ubuntu-core-troubleshooting>` — covers the common boot and install issues with known fixes.
- {ref}`Use a recovery mode <how-to-guides-manage-ubuntu-core-use-a-recovery-mode>` — for getting into a working state when normal boot fails.
