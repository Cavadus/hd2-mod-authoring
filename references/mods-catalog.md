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
| TacticalCombatOverhaul v2.4.1 (rojo) | mods/rojo/tco_* (7 addons) | stamina/recovery/aim-sway/exhaustion patched via raw ffi ReadProcessMemory + stingray; reuses the DRIVER HUD network/inventory offsets (0x346BF98, 0x3326738, 0xF22EC8, 0xF1AEB0, 0xF32F18). Shows a whole game system we hadn't touched |
| WarbondDesk (warbond_desk) | mods/warbond_desk/main | NATIVE CODE patching: x64 byte-signature scans (hex + mask + anchor + reference_offset + kind="code") locate and patch game .text instructions, with image_base_offset/table_offset/table_count to resolve RIP-relative tables from code. A technique entirely different from value-table patching |
| Bolter-Jar Dominator (david_hogins) | mods/david_hogins/bolter_jar | the Dominator rechamber done via HD2Runtime typed API (`hd2.options` menu + `require('mods/skyeshade/hd2runtime')` + explosion-demolition choice) — confirms the typed route for our exact use case |
| Wrench-Repair 1.3.0 (ymir) | mods/ymir/entrenchment_repair | shovel contact repair via HD2Runtime; adapted from Field Repair 1.3 |
| Support Side Holster 2.6.1 (local) | mods/local/holster_* (67 addons) | multi-option architecture: dozens of tiny 200-byte addons each set one weapon hash in a shared `_G.SupportHolsterOptions` table, one big worker reads them |
| FirstPerson v2.9 (jim) | mods/jim/first_person | stingray camera repositioning (head node + offset, sr.Vector2/Vector3, ActionCam-style); no stat memory writes |
| SmoothBoot 3.0 (codex) | mods/codex/smoothboot | a rule engine that re-heads the Bingus mod chain, throttles per-mod (trip_ms circuit breaker), attributes errors by chunk name — meta-infra, not game data |
| Overdive v1.0 (hd2lab) | mods/hd2lab/spawn_director | spawn-director Lua is OBFUSCATED/encrypted (only the addon header line is readable) — some mods ship unreadable payloads; can't mine technique |
| Transmog Probe (cow) | mods/cow/transmog_probe | embeds a pre-rasterized glyph atlas (rects) + geometry probe — same glyph-atlas technique as Mech Part HUD |
| Mission Reroller (ipodalexei) | mods/ipodalexei/mission_reroller* | development core with SUPPORTED_BUILD + record layouts + inspect_snapshot; no native adapter yet |
| Enemy Balance & Behavior / Gameplay Systems 1.2 (enemyov) | mods/enemyov/** (64 addons) | ENEMY ATTACK REFERENCE MECHANISM — the missing piece behind the Bile Titan bug. Enemy attacks reference projectiles through a SEPARATE enemy-weapon component `0xd25fc7f7` (stride 1232 — same stride as player WeaponData 0x88E4DBB1, but a different type hash), NOT through the player `ProjectileWeapon` (0x45171B68). Attack schema: weapon_type 0xd25fc7f7 (1232), spread_type 0xf916ed4b (12), component_type 0xb4789330 (44), damage_type 0x260cbe2b (76, root 0xe0a72cf0). Enemy actors are keyed by 32-bit hash (e.g. bile-spewer head 0x8c5570c9, resource 0xccae5264acd591b7) with named zones; HealthComponent zones carry armor at +216 (matches our exosuit armor offset). Uses a native-code-patching framework (RIP-relative memory resolution + code disassembly) — same class as WarbondDesk |
| GL-15 Evictor Ammo Selector (working) | — | hand-rolled programmable-ammo reference mod for the Evictor |
| Super-Earth Armory Forge (community) | mods/community/passive_picker_v4 | passive-picker UI |
| Weakpoint LockOn All-in-One | (19 binary) | enemy weakpoint targeting — binary patches, not Lua |

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
