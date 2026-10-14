# omarchy-legacy

**Unofficial.** Not affiliated with, or endorsed by, Omarchy, DHH or Basecamp.
Upstream project: [github.com/omacom/omarchy](https://github.com/omacom/omarchy) · [omarchy.org](https://omarchy.org)

This repository keeps the **Omarchy 3.8.4 online installer** working on an existing Arch Linux system, so you can install Omarchy manually (for example on a separate partition, or alongside GRUB) and then upgrade to Omarchy 4 "Quattro".

In October 2026 upstream removed its `master` branch, the `omarchy.org/install` script started returning 404, and the `omarchy-walker` package was retired. Together those broke every manual 3.x install. This repo is a snapshot of upstream tag `v3.8.4` (commit `8fcc9d6`) with the smallest set of changes needed to make it install again.

> **Use this as a bridge, not a destination.** Omarchy 3.x no longer receives fixes, including security fixes. Upgrade to Omarchy 4 right after installing (see below).

Full walkthrough: [musabase.com](https://www.musabase.com)

## Install

On a fresh Arch Linux install, logged in as your normal (non-root) user:

```bash
curl -fsSL https://raw.githubusercontent.com/musabase/omarchy-legacy/legacy-3.8.4/boot.sh | bash
```

The installer may warn that it requires the Limine bootloader or a Btrfs root filesystem. On a GRUB or systemd-boot system, answer **Yes** to "Proceed anyway". This repo's changes let the install finish without Limine.

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

Nothing else is modified. Compare against upstream with:

```bash
git diff legacy-3.8.4~1 legacy-3.8.4
```

## Known limitation

Only the **installer scripts** are frozen here. Packages still come from Omarchy's own servers (`pkgs.omarchy.org`, `stable-mirror.omarchy.org`), which now serve Omarchy 4. If upstream retires another package that 3.8.4 asks for, the install can break again until this repo is patched. Please open an issue if that happens.

## Upgrade to Omarchy 4 (Quattro)

After the install finishes and you've rebooted into the Omarchy desktop, open a terminal and run the official upgrade script:

```bash
curl -fsSL https://raw.githubusercontent.com/omacom/omarchy/quattro/bin/omarchy-upgrade-to-quattro -o ~/upgrade-to-quattro
bash ~/upgrade-to-quattro
```

The upgrade is one-way. Take a backup or a VM snapshot first.

### Extra steps on GRUB systems

Do these **before** running the upgrade script:

```bash
# 1. Remove any Limine stubs you created during the install
sudo rm -f /boot/limine.conf /usr/local/bin/limine-update /usr/local/bin/limine-mkinitcpio

# 2. Skip the Limine-only "install the Omarchy kernel" migration
#    (install linux-omarchy yourself afterwards, see below)
sudo install -Dm644 /dev/null /var/lib/omarchy/migrations/1789325478

# 3. Drop the btrfs-overlayfs initramfs hook, which only ships with Limine's packages
sudo tee /etc/mkinitcpio.conf.d/zz-grub-no-overlayfs.conf >/dev/null <<'EOF'
if [[ ! -e /usr/lib/initcpio/install/btrfs-overlayfs && ! -e /etc/initcpio/install/btrfs-overlayfs ]]; then
  _grub_hooks=()
  for _grub_hook in "${HOOKS[@]}"; do
    [[ $_grub_hook == btrfs-overlayfs ]] || _grub_hooks+=("$_grub_hook")
  done
  HOOKS=("${_grub_hooks[@]}")
  unset _grub_hooks _grub_hook
fi
EOF
```

When the upgrade script asks **"Reboot to complete Quattro upgrade now?"**, answer **No**, then:

```bash
sudo mkinitcpio -P
sudo grub-mkconfig -o /boot/grub/grub.cfg
reboot
```

### Optional: the Omarchy kernel

Omarchy 4.0.4 made its own `linux-omarchy` kernel the default. On GRUB, install it yourself once Omarchy 4 boots:

```bash
sudo pacman -S linux-omarchy linux-omarchy-headers
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

## License

Omarchy is released under the [MIT License](LICENSE), Copyright (c) David Heinemeier Hansson. The changes in this repository are released under the same license.
