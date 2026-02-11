# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

**bali** — Personal Arch Linux installer for Dell laptops with i3wm. Single bash script (`letsgo`), validated by Goss (`goss.yaml`).

## Commands

```bash
./letsgo          # Run the installer (requires sudo, internet, fresh Arch)
goss validate     # Validate installed packages
```

## Critical Gotchas

- **CWD trap:** `cd yay` (line 16) permanently changes the working directory. The script never returns to the original directory. All subsequent commands happen to be path-independent (`yay -S`, `systemctl`), but any new code using relative paths after line 17 will break. Add `cd -` or use absolute paths.
- **Not fully non-interactive:** `makepkg -si` (line 17) and `sudo pacman -Syyu` (line 12) require user confirmation. Only the `yay -S --noconfirm` commands are non-interactive.
- **Not idempotent:** `git clone yay.git` (line 15) fails on re-run if the `yay/` directory already exists.
- **No error handling:** No `set -e` or `set -o pipefail`. A failed package install does not stop subsequent installs.
- **Dell-only hardware commands:** `dell-bios-fan-control 0` (line 69) modifies BIOS settings directly and will fail/behave unpredictably on non-Dell hardware. `i8kutils` and `i8kmon` are Dell-specific.
- **rfkill masked:** `systemctl mask systemd-rfkill.service systemd-rfkill.socket` (line 70) permanently disables RF kill switch management. Hardware airplane mode toggle may stop working.
- **Slow step:** `pacman-key --refresh-keys` (line 9) can be very slow depending on keyserver availability.

## Hardcoded Personal Config

- Timezone: `America/Sao_Paulo` (line 25)
- Dell fan/thermal management throughout (lines 65-74)
