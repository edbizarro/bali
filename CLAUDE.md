# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

**bali** — Personal Arch Linux installer for Dell laptops with i3wm. Modular bash installer (`letsgo`) with external package lists (`packages/*.txt`), validated by Goss (`goss.yaml`).

## Project Structure

```
letsgo              # Main installer script (227 lines)
packages/           # Package lists (one category per file)
  base.txt          # openssh, git
  datetime.txt      # ntp
  connectivity.txt  # Network & bluetooth (10 packages)
  terminal.txt      # Shell utilities, monitoring, file mgmt (28 packages)
  dev.txt           # Dev tools, cloud CLI, editors (8 packages)
  productivity.txt  # Browser, office, backup, archiving (10 packages)
  ricing.txt        # i3wm, polybar, rofi, picom, theming (16 packages)
  fonts.txt         # Nerd fonts and system fonts (22 packages)
  display.txt       # GPU drivers, display management (9 packages)
  power.txt         # Dell-specific thermal/fan/power (10 packages)
  audio.txt         # Pipewire, mopidy, music tools (24 packages)
  touchpad.txt      # libinput drivers (2 packages)
goss.yaml           # Package validation tests
```

## Architecture

The script is structured as:

1. **Helper functions** — `read_packages()` parses `.txt` files (strips comments/whitespace), `install_packages()` installs with per-package success/failure tracking, `print_report()` generates a formatted summary table.
2. **System bootstrap** — `pacman -Syy`, system update, base packages (pacman only, yay not yet available).
3. **Yay installation** — Conditional (`command -v yay`), clones into a temp directory via subshell.
4. **Package sections** — Sequential `install_packages` calls for each category.
5. **Post-install config** — NTP, timezone, network detection, Dell BIOS/fan/thermal services.
6. **Report** — Tabular summary with failure details and log file path.

Error tracking: failed packages are logged with section, name, and extracted error message. Full log written to `/tmp/bali-install-YYYYMMDD-HHMMSS-XXXXXX.log`.

## Commands

```bash
./letsgo          # Run the installer (requires sudo, internet, fresh Arch)
goss validate     # Validate installed packages
```

## Critical Gotchas

- **Fully non-interactive:** All pacman/yay/makepkg calls use `--noconfirm`. No user prompts during install.
- **Partially idempotent:** Yay installation checks `command -v yay` and skips if present. However, some post-install `systemctl enable` commands may warn on re-run.
- **Error handling is partial:** `set -o pipefail` is set and failures are tracked/reported, but the script does not abort on failure — all sections run regardless.
- **Dell-only hardware commands:** `dell-bios-fan-control 0` (line 202) modifies BIOS settings directly and will fail/behave unpredictably on non-Dell hardware. `i8kutils` and `i8kmon` are Dell-specific.
- **rfkill masked:** `systemctl mask systemd-rfkill.service systemd-rfkill.socket` (line 203) permanently disables RF kill switch management. Hardware airplane mode toggle may stop working.
- **pacman-key refresh disabled:** The `pacman-key --refresh-keys` call is commented out (lines 123-124). Uncomment if you hit key validation errors, but expect it to be slow.
- **Package lists are the source of truth:** To add/remove packages, edit the `.txt` files in `packages/`. Comments (`#`) and blank lines are supported.

## Hardcoded Personal Config

- Timezone: `America/Sao_Paulo` (line 158)
- Dell fan/thermal management (lines 198-208)
