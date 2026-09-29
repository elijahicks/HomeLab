# Installation: Base OS

## Pre-install

1. Identified the exact hardware model via **About This Mac** and
   confirmed the absence of an Apple T2 security chip (via the
   presence of "Startup Disk" rather than "Startup Security Utility"
   in Recovery Mode) — relevant because T2-equipped Macs require
   Secure Boot to be lowered before booting non-Apple installers.
2. Backed up any data on the target drive.
3. Erased the internal drive from Recovery Mode Disk Utility at the
   top-level physical disk (not just the individual APFS volumes),
   to leave a completely clean disk for the new OS.

## Installer media

- Downloaded a Linux Mint ISO and flashed it to a USB drive with
  **balenaEtcher**. Note: copying an `.iso` file onto a USB drive via
  drag-and-drop does not produce a bootable drive — the image must be
  written at the block-device level.
- Booted the target machine from the USB via the EFI boot picker
  (held **Option** during startup).

## Installation

- Standard guided install; selected "Erase disk and install" since the
  disk was already wiped.
- Enabled third-party multimedia codecs during setup (required for
  video playback later).
- Created the primary service account and set the hostname.

## Post-install configuration

- Retained the Cinnamon desktop environment (rather than switching to
  a headless `multi-user.target` boot target) to preserve local
  troubleshooting capability via an attached display, at the cost of
  some idle resource overhead.
- Replaced the default terminal emulator with **Tilix** for
  tiling/split-pane support: `sudo apt install tilix`.
- Reserved a static IP for this host at the router level (DHCP
  reservation by MAC address), so all subsequent service configuration
  can rely on a fixed address.
