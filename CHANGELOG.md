# Changelog

All notable changes to the `hd2-mod-authoring` skill are documented here.
The format follows [Keep a Changelog](https://keepachangelog.com/), and the
version matches the ClawHub release (`openclaw skills install @Cavadus/hd2-mod-authoring`).

## [0.1.6] — 2026-10-07

### Added
- **Derived entity hashes.** Keyed weapon writes read the entity hash out of the ProjectileWeapon bucket after a content match. A multi-weapon pack matches the ProjectileSettings round by name hash plus speed, mass, drag, gravity, arming, and damage type, not by the projectile type number alone. A round fired by several weapon records still counts if one field the mod does not write narrows them to one survivor. Zero or two survivors means write nothing, and the seed hash is only a drift log. Stash WeaponData and LoadoutEntry until both projectile tables have been seen. Realistic Weapons 1.3 is the worked case: the One-Two grenade is shared by two records, the MG-206 round by five.
- **Deploy is a different file from the mod folder.** Arsenal writes the enabled patches into `Helldivers 2/data/9ba626afa44a3aa3.patch_0` only on deploy. Bingus reads that file once, at process start. Quit, deploy, then launch, and confirm the new version string is in the data file. An option can be checked while the mod record's own `enabled` is false; the option does nothing then.
- **Fragility order** for "will an update break this": content identity, content-anchored value scan, entity-hash lookup, fixed code offset. A fail-safe lookup that no-ops is still broken.
- **HD2Runtime output map.** One output-family component per weapon (projectile, beam, arc, spray, melee), status slots on the DamageInfo row matched by name, the package-residency ceiling on cross-weapon projectile swaps, and fire modes as a 4-slot list with quaternary at +156.
- **Firing-sound loudness.** A first-shot-louder report is in the bank, not the Lua. Measure Wwise Vorbis with vgmstream. Splice a quieter donor WEM. The Dominator bank rebuild (three length fields, preserve the unidentified WEM hash chunk) is in `references/install-and-format.md`. PCM in a Vorbis-declared slot is an unverified experiment.
- **Thumbnail redesign** procedure in `references/thumbnail.md`. A one-line tweak edits the shipped 2048×2048 file; a redesign rebuilds from the clean cutout.
- **Census join.** diag-5 already writes `refscan_pwbuckets.txt`. Run the census with only refscan loaded. `refscan_enemy.txt` still does not appear. The `name==0` enemy bias is the measured 3.25× rate.
- **Tactical Combat Overhaul 2.6.13 and Wrench Repair 1.3.0** reread into `references/mods-catalog.md`. Reload and fire rate are copied at weapon build. WeaponData ergonomics at +356 is not live handling. Settings pages are often read-only. Mod Options Menu drops a mod past 32 options. Hull health is two copies.

## [0.1.5] — 2026-10-06

### Added
- **Safety boundary.** Runtime memory patching is limited to a game process you started, on a machine you own. Read before write, roll back on mismatch, and do not reuse the technique on other software or other people.
- `CHANGELOG.md` in the published package. This entry backfills the ClawHub 0.1.5 release, which went out without a local changelog section.

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
