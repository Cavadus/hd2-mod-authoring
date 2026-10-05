# Mod catalog (scanned the mods archive, 2026-10-02)

Classification of every archive, by addon id and kind. "Maintained" = kept in the source tree; "mined" = read for a technique; "cataloged" = classified, addon id recorded.

## Maintained (worked examples)

| Mod | Addon | Kind |
|---|---|---|
| JAR-5 Dominator: Rechambered | mods/dsh/dominator_bolt_pistol | lua |
| Mech Part HUD | mods/recon/mech_part_hud (was mods/arena_local/mech_part_hud) | lua |
| Exosuit Rebalance (ammo + armor + EAT rockets) | mods/recon/exosuit_ammo | lua |
| Exosuit Armor +1 (folded into Exosuit Rebalance; do not install separately) | mods/recon/exo45_armor | lua |
| M-1000 (AP4 + free movement) | mods/maxigun/ap4_public, mods/maxigun/free_movement_public | lua |
| Realistic Weapons v1.1 | mods/recon/realistic_weapons (flags: mods/recon/realistic_weapons_*) | lua |
| Dixie Horn | (Wwise bank, no addon id) | wwise_bank |

The SHODAN tuners (mods/shodan/{arbitrator,onetwo,adjudicator,evictor,mg206}_tuning)
are the pre-merge Realistic Weapons; retired.

## Third-party, mined for a technique

| Mod | Addon | What it gave us |
|---|---|---|
| SHODAN Stat Editor (github SHODAN-HORAI) | mods/shodan/stat_editor | cross-confirms projectile velocity at +32, type 0xBD4042C2, stride 272 (== Dominator OFF_SPEED); writes only on edit/start/map-reload, no re-assert loop |
| JAR-5 Buff Pack v2.12 (joelc) | mods/joelc/jar5_buff_* | fire modes are a u32 enum array in a ~144-byte record (anchors 0x42480000/0x42F00000 at +0/+4, modes at +0x70/+74/+78, TertiaryFireMode +0x7C None(0)->Auto(1)); reload is a passive stat (type 0x63CE0FEB, stat id 13, f32 1.36) repointing every armor passive block, not a weapon field. Damage/AP/speed patches cross-confirm DamageSettings 0xE0A72CF0 (damage +4, AP +12) and ProjectileSettings 0xBD4042C2 (speed f32 +32, name_upper +4 = 0x1E2FAF6F). NOTE: uses the OCLAW_UPDATE_BUS dispatcher — same attribution/JIT caveat as Realistic Weapons (SKILL section 7); heavy full-address-space scanner |
| EXO-45 Patriot Buff | mods/rexsybimatw/exo45_patriot_buff | HealthComponentData layout + byte-edit pattern |
| Exosuit Heavy Armor | mods/recon/exo45_armor | HealthComponentData value-scan + identity guard |
| DRIVER HUD 1.5.1 (installed) | — | network/inventory offsets for the equipped-weapon read |

## Third-party, cataloged (techniques observed)

| Mod | Addon(s) | Notes |
|---|---|---|
| SHODAN Stat Editor 简体中文补丁 | mods/shodan/stat_editor_zh_cn | translation layer, NOT a stat writer: hooks Gui.text/text_extents only, zero memory writes; manifest Description is "需要在原版之后加载" (load after the original editor) |
| Armored Overhaul 2.0.1 | mods/chef/armored_overhaul_* (12 addons) | vehicle handling; config-file + loader-api; uses 0x6AB3B43F game.dll anchor |
| Field Repair (ammo only) | mods/combat/field_repair, mods/combat/field_repair_ammo | patches WeaponMagazineComponentData (0xFB8D88A3); config-file |
| Smarter Guard Dogs & Sentries | mods/chef/smarter_guard_dogs_* (11 addons) | AI behavior; config-file |
| Vanilla Plus Megapack v14/v16 | mods/cowboybingus/* (8 addons) | CowboyBingus's own; loader-api + config-file |
| GL-15 Evictor Airburst | mods/dsh/evictor_gas_airburst | function_info.left = ProgrammableAmmo (known) |
| LAS-98 AP Ramp | mods/ymir/laser_cannon_ramp | DamageSettings (0xE0A72CF0) |
| MS-11 Solo Silo | mods/codex/solo_silo_demo50 | DamageSettings (0xE0A72CF0), dlsum(DamageSettings) |
| EXO Stratagem Launcher | mods/codex/exo_launcher | MurmurHash64A (0xC6A4A7935BD1E995) |
| Casemate Turrets | (binary) | type 0xe0a48d0be9a7453f |
| FRV Anti-Flip | (binary) | type 0x5f7203c8f280dab8 |
| Bingus Shared Loader v15 | (loader) | the framework itself |
| WWSC All-in-One | (rar) | sound banks; the Dominator SM2 firing sound came from here |

## Reusable constants surfaced by this scan

- `0xFB8D88A3` dlsum("WeaponMagazineComponentData") — magazines/ammo.
- `0xE0A72CF0` dlsum("DamageSettings") — damage type table (damage/durable/AP per type).
- `0xB3915DE3` dlsum("HealthComponentData") — exosuit health + armor.
- `0xC6A4A7935BD1E995` MurmurHash64A multiplier (the packer's hash); low half 0x5BD1E995.
- `0x6AB3B43F` a game.dll layout anchor (appears in multiple mods and the Mech Part HUD game.dll pair).
- The config-file pattern: `%LOCALAPPDATA%` key=value overrides read at load, so tunables change without repacking. The Dominator `read_overrides` and many third-party mods use it.
