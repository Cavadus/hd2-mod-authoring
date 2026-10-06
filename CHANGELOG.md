# Changelog

All notable changes to the `hd2-mod-authoring` skill are documented here.
The format follows [Keep a Changelog](https://keepachangelog.com/), and the
version matches the ClawHub release (`openclaw skills install @Cavadus/hd2-mod-authoring`).

## [0.1.4] — 2026-10-06

### Added
- **Clone-safe projectile overwrite technique.** Documented how to prove a
  projectile record is "referenced by nothing" three ways (weapon slots +0/+576,
  WeaponRounds +64, shrapnel +84, and enemy attacks) before overwriting it — with
  the statistical `name==0` proof that enemy projectiles carry no localisation
  name. Census-first discipline: dump every LDLD table, then pick a genuinely
  dead record.
- **Reference diagnostics as worked examples.** `examples/refscan` (read-only
  LDLD census) and `examples/fmscan` (fingerprint value-scan) — two ready-to-copy
  Bingus addons demonstrating the memory-inspection techniques, each with its own
  manifest and packer.

### Fixed
- **Cross-manager manifest compatibility.** Arsenal, DDMM, and teutinsa's HD2 Mod
  Manager all read the same V1 `manifest.json`; spaces in `Include` paths are
  legal everywhere; the real "invalid include path" failure is Arsenal's Mods
  Storage Folder pointed at a stale/transient Windows `Temp` path, not a manifest
  bug.

## [0.1.3] — 2026-10-06

### Added
- **Clone-safe technique.** Prove a projectile record is "referenced by nothing"
  before cloning it — the three-way check (weapon slots + shrapnel/re-arm + enemy
  `name==0` rule) with the `refscan` read-only census diagnostic.
- **New reference `references/memory-census.md`.** LDLD table format, the three
  dump bugs that each cost a game relaunch, that enemy data is NOT a standalone
  table, and the reference-set census.

## [0.1.2] — 2026-10-05

### Removed
- `skill-card.md`. No other changes to source or documentation.

## [0.1.1] — 2026-10-05

### Removed
- Non-critical files: `LICENSE`, `README.md`, `skill-card.md`, and `thumbnail.jpg`.

### Changed
- Updated skill metadata with homepage information.
- Expanded the description and documentation to cover static vetting of
  downloaded mod managers/tools for malware.
- Clarified that some mods (such as exosuit-ammo) are local-only and not
  distributed with this skill.
- Improved workflow instructions and safety practices for mod installation,
  editing, and verification.

## [0.1.0] — 2026-10-05

### Added
- Initial release of `hd2-mod-authoring` with mod install, classification, and
  patch management procedures.
- Workflows for identifying patch types (Lua, Wwise, archive, or language).
- Steps to check, avoid, and resolve conflicts between mods that write the same
  game value.
- Instructions for editing mods, packing, backup management, and version
  consistency.
- Methods for glyph-atlas HUD translation and guidance for sound and HUD overlay
  mods.
- Best practices for memory patching, scanning discipline, and offset
  verification after game updates.
