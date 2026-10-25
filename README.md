# omarchy-legacy

**Unofficial.** Not affiliated with, or endorsed by, Omarchy, DHH or Basecamp.
Upstream project: [github.com/omacom/omarchy](https://github.com/omacom/omarchy) · [omarchy.org](https://omarchy.org)

This repository keeps the **Omarchy 3.8.4 online installer** working on an existing Arch Linux system, so you can install Omarchy manually (for example on a separate partition next to Windows) and then upgrade to Omarchy 4 "Quattro".

In October 2026 upstream removed its `master` branch, the `omarchy.org/install` script started returning 404, and the `omarchy-walker` package was retired. Together those broke every manual 3.x install. This repo is a snapshot of upstream tag `v3.8.4` (commit `8fcc9d6`) with the smallest set of changes needed to make it install again.

> **Use this as a bridge, not a destination.** Omarchy 3.x no longer receives fixes, including security fixes. Upgrade to Omarchy 4 right after installing (see below).

**Full walkthrough** (Btrfs setup, the Snapper fix before upgrading, the upgrade itself, and adding Windows to Limine, with screenshots): [How to Install Omarchy 4 on a Separate Partition (Dual Boot)](https://www.musabase.com/2026/06/install-omarchy-os-manually-separate-partition.html)

Tested in October 2026: Omarchy 3.8.4 from this repo, upgraded to Omarchy 4, with Windows 10 on its own disk and both booting from Limine.

## Install

On an Arch Linux install with a **Btrfs** root, logged in as your normal (non-root) user:

```bash
curl -fsSL https://raw.githubusercontent.com/musabase/omarchy-legacy/legacy-3.8.4/boot.sh | bash
```

The installer warns that it requires the Limine bootloader. Answer **Yes** to "Proceed anyway". This repo's changes let the install finish without Limine, and Limine arrives later with the Omarchy 4 upgrade. Btrfs is required: the upgrade sets up Snapper, which needs it.

If the install stops, choose **Retry installation** from the menu. Do not run the curl command again: it deletes `~/.local/share/omarchy` and starts over.

## What's changed from upstream v3.8.4

| File | Change | Why |
|---|---|---|
| `install/omarchy-base.packages` | `omarchy-walker` → `walker` | `omarchy-walker` and the Elephant packages were removed from Omarchy's package repo on 2026-10-08. `walker` itself is still published. |
| `install/first-run/elephant.sh` | Made a no-op | Elephant is no longer installable, and its failure would skip the rest of the first-login setup. |
| `install/login/limine-snapper.sh` | Limine boot-entry check only runs when Limine is installed | Upstream aborts when `/boot/limine.conf` is missing, which is always the case on GRUB/systemd-boot. |
| `boot.sh` | Defaults to this repo and the `legacy-3.8.4` branch | Upstream defaulted to the deleted `master` branch. |
| `bin/omarchy-reinstall-git` | Clones this repo instead of upstream `master` | Same reason. |
| `README.md` | This file (the original is kept as `README.upstream.md`) | |
| `quattro/omarchy-upgrade-to-quattro` | Added: unmodified copy of upstream's upgrade script at commit `2fca7fe` | Fallback in case the upstream script is moved or removed. |

Nothing else is modified. Compare the installer changes against upstream with:

```bash
git diff 4b172ef c554330
```

(`4b172ef` is the untouched upstream v3.8.4 import, and `c554330` is the commit with the installer changes.)

## Known limitation

Only the **installer scripts** are frozen here. Packages still come from Omarchy's own servers (`pkgs.omarchy.org`, `stable-mirror.omarchy.org`), which now serve Omarchy 4. If upstream retires another package that 3.8.4 asks for, the install can break again until this repo is patched. Please open an issue if that happens.

## Upgrade to Omarchy 4 (Quattro)

### Before you upgrade

The upgrade is one-way, so take a backup first.

If `findmnt /.snapshots` prints a line, you already have a `.snapshots` subvolume, and the upgrade's Snapper setup will fail until you apply a short fix. The commands are in the [full guide](https://www.musabase.com/2026/06/install-omarchy-os-manually-separate-partition.html#prepare-quattro-upgrade).

### Run the upgrade

After the install finishes and you've rebooted into the Omarchy desktop, open a terminal and run the official upgrade script:

```bash
curl -fsSL https://raw.githubusercontent.com/omacom/omarchy/quattro/bin/omarchy-upgrade-to-quattro -o ~/upgrade-to-quattro
bash ~/upgrade-to-quattro
```

If that upstream URL stops working, use the copy saved in this repo (upstream commit `2fca7fe`):

```bash
curl -fsSL https://raw.githubusercontent.com/musabase/omarchy-legacy/legacy-3.8.4/quattro/omarchy-upgrade-to-quattro -o ~/upgrade-to-quattro
bash ~/upgrade-to-quattro
```

### After the upgrade

Answer **Yes** to "Reboot to complete Quattro upgrade now?". After the reboot, the Limine menu replaces GRUB. Don't run `grub-mkconfig` from here on. To add Windows to the Limine menu:

```bash
sudo limine-scan
```

Pick Windows Boot Manager by its number. BitLocker notes, troubleshooting and screenshots are in the [full guide](https://www.musabase.com/2026/06/install-omarchy-os-manually-separate-partition.html#add-windows-to-limine).

## License

Omarchy is released under the [MIT License](LICENSE), Copyright (c) David Heinemeier Hansson. The changes in this repository are released under the same license.
