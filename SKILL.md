---
name: hd2-mod-authoring
description: "Helldivers 2 mod, Arsenal install, Bingus lua patch, Wwise sound-bank swap, Nexus unlink, an HD2 Mod Manager install that dies on the manifest Name, translating on-screen text in a glyph-atlas HUD, or drawing a live HUD overlay with the game font: classify the install, conflict-check writers of the same value, then edit, pack, and verify from the game log."
---

# HD2 mod authoring

Work from `~/clawd/hd2-mods/<mod>/`. The live install is `~/.config/hd2arsenal/mods`. Offsets and the 2026-10-01 overlap list are in `references/install-and-format.md`. Re-read that file at the step that cites it. Re-check any layout a game update could move.

## 1. Classify the install before editing

List `~/.config/hd2arsenal/mods`. For each `.patch_0`, find the payload. A Lua patch starts at byte 200. A Wwise bank's `BKHD` is later (the Dixie bank starts at 320). Search for `BKHD` before treating byte 200 as the payload.

- Lua runtime patch: the bytes at 200 contain `HD2-Addon:` or a Lua `function`, and the type at `0x50` is the Lua type in the reference. These need Bingus Shared Loader.
- Binary resource patch: no `HD2-Addon:` and no Lua `function`, or the file contains `BKHD`. `.gpu_resources` and `.stream` sidecars belong to archive patches. Copy those bytes. Do not pack them as Lua.
- The same `HD2-Addon:` id in two mod folders means both versions are installed. Different ids still stack when they write the same field.
- A language patch is a Lua patch that edits nothing. Recognize it before treating it as a stat editor: addon id ends `_zh_cn` (or the manifest `Name` carries `简体中文`), the `Description` says it loads after the original ("需要在原版之后加载"), and the body only hooks `stingray.Gui.text` / `text_extents` (draw/measure) with zero `WriteProcessMemory` / `ReadProcessMemory`. It cannot fight a value writer — the real editor is a different addon (`mods/shodan/stat_editor`).

Done when the reply names the patch kind and any duplicate addon id.

## 2. Refuse a second writer of the same value

Read the overlap list in `references/install-and-format.md`, then re-scan. A folder under `mods/` is only the library. What loads is `hd2a_data.json` → `modsList[<selectedProfile>].mods` with `enabled: true`, and which suboption is on. The Patriot and Lumberer ammo pairs were library-only; they were not in the profile. Exosuit Rebalance (`mods/recon/exosuit_ammo`) is the writer for those magazines and for armor (+1); the standalone `exo45_armor` was folded into it. Do not put the ammo mods or the standalone armor mod back — armor would double-apply. Bingus does not dedupe different addon ids.

Name the installed mod that already writes the field. Stop unless the operator asked to replace it. To remove one, move the folder to `~/clawd/hd2-mods/_retired/` and delete that uuid from both `modsLibrary` and the profile. Copy `hd2a_data.json` aside first. It is indent-2 JSON.

Done when the reply names the overlap, or says there is none.

## 3. Edit the source tree, then make the text agree

Our mods:

- `hd2-mods/dominator-rechambered/` — addon `mods/dsh/dominator_bolt_pistol`, source `jar5_dominator_rechambered.lua`.
- `hd2-mods/exosuit-ammo/` — addon `mods/recon/exosuit_ammo`, source `exosuit_ammo.lua`. Personal. Do not upload it.
- `hd2-mods/m1000/` — addon `mods/recon/m1000`, source `m1000.lua`. Arsenal folder `m1000_combined`. Gameplay is one scan. Sights and sounds stay the original archive patches.
- `hd2-mods/realistic-weapons/` — one scanner, addon `mods/recon/realistic_weapons`. The weapon checkboxes are flags, not scanners. Ids are in the reference.
- `hd2-mods/dixie-horn/` — Wwise bank, not Lua. No `REVISION`. The bank procedure is in the reference.
- `hd2-mods/mech-part-hud/` — read-only glyph-atlas HUD, addon `mods/recon/mech_part_hud`. Not a writer of health or ammo. A spatial arm link stays; do not wipe it every poll. The label procedure is below; the fit and the walk are in the reference.

### Translating on-screen text

Decode as UTF-8 before counting CJK. A UTF-16LE decode of ASCII reports Chinese in every Lua file. That scan was wrong.

Grep the draw function for strings it passes to the glyph renderer. A `zh` field or label table that nothing reads is not on screen.

If those glyphs are pre-rasterized rects, the generator charset is the only letters that draw. Regenerate the font section for the new strings. The silhouette bytes before `["fonts"]` must stay identical. Measure the label against the panel first. The Mech Part HUD fit, and the fact that its `tools/build.py` imports `lupa` (not installed), are in `references/install-and-format.md`. Pack with that mod's `archive_writer.py` and check the assembly with `/tmp/luavenv/bin/luaparser`.

Replacing the `.patch_0` leaves `README.md` and `Source/` that Arsenal unpacked beside it. Overwrite those from the new package. A copyright name inside a bundled license is not a label.

Done when the installed patch and the installed README have no CJK in a UTF-8 decode, luaparser exits 0, and the silhouette prefix matches. `deployed: false` means the panel has not been seen in game.

Drawing a live overlay with the game font (not the glyph atlas) and reading game-object state are in `references/hud-overlay.md`: the `Gui.text` recipe, the `sr.Vector2` callable-table and `App.worlds` pitfalls, and the `game_object_field` unknown-name hang.

Before another behavior edit, read the game log named in the source. Its revision line must already match the build you believe is loaded. A previous revision means Arsenal moved the folder or the game did not restart.

Copy the current Lua to a backup before editing, and keep old versioned sources and superseded zips in an `archive/` subfolder inside that mod folder (not the mod root or a shared top-level pile). Name the backup `.v<old-revision>.lua.bak`. Change `REVISION` only when that behavior is the one being shipped. A rejected experiment keeps the last shipped `REVISION` and puts a distinct token in the log line the source already prints, so the log proves which build ran. Do not mint the next release number per test. Current Dominator release numbers are in `references/install-and-format.md`. A ricochet experiment does not get the next number. The close-range stop is still unsolved. Make `manifest.json` describe the shipped behavior, not the experiment. If the opening comment disagrees with `REVISION` and the manifest, fix the comment in the same edit. The Dominator comment has claimed "original range" after the range cap changed. `MEMORY.md` still says dbp-1.1. Trust `REVISION` and the manifest.

Done when the backup exists and `REVISION`, the manifest, and the opening comment match.

## 4. Patch memory the way the failed builds taught

Guard the file with `rawget` / `rawset` on one global so it loads once.

Never scan process memory in one frame. One scan, then set `state.done`. `maintain` is one read-back of the cached address, not another scan. A full rescan every few seconds is what put the Dominator on Mod Lag Watchdog.

The same discipline applies to locating a record inside an already-found table on a
hot path. Cache the record's address keyed on the table's identity (the records
pointer or block address), and re-validate with one field read (e.g. the type u32 at
+0); only re-walk the table when that identity changed (map reload). The Dominator
wheel's `ammo_apply` runs ~10×/sec and used to re-walk all ~350 projectile records
every call to re-find the alternate round it had already located — caching the
address (keyed on the table's records pointer) and re-validating with one type read
cut it to a single 4-byte read per call. Cue: a per-cycle `for i = 0, count-1`
searching a table you already located.

Anchor only on vanilla values. Never match a record by the value this mod writes. Compare before write, read the bytes back, and restore page protection.

Re-verify offsets after a game update. The layouts checked against the game data, and how to find a component type for a field you have not patched before, are in `references/install-and-format.md`.

- Emancipator autocannon is the exosuit branch that must not require `magazines_max == 0`. The other four weapons do. Requiring max 0 for all of them drops the autocannon.
- Patriot missiles are two identical capacity-14 records. `MISSILE_PICK` writes one. If the Patriot still shows 14, flip it.
- Do not add a fire-mode or `fire_ability` write. Those values are cached when the weapon is built, and live writes did not change the Dominator's burst. The stock fire-mode selector already bursts. Arming distance (+160) is read per shot, not cached — which is why a live write works. The wheel's alternate (section 11) is the P/40-K's own round (0.1 m arming); the Dominator's own round keeps `GYRO_ARMING` (12 m).
- A projectile-field patch does not change the armory's AP or Explosive label. The
  label is a **presentation** field, not native code: it is writable through
  HD2Runtime `hd2.fields.presentation.armor_penetration` / `presentation.traits`,
  backed by `LoadoutEntryComponentData` +12 (section 10). Flagged
  `allow_unverified_effect`, and menus build their labels when they open.
- **Impact sound is not a projectile-record field either.** It resolves through
  `surface_impact_type` (+168) / `ricochet_impact_type` (+172) — `SurfaceImpactType`
  enum values — into a separate `PhysicsImpactEffectComponentData` lookup whose
  `PhysicsImpactEffectInfo` carries `audio_effect_id`, `camera_effect_id`,
  `shake_effect_id`, `decal_sheet`, `quake_effect`. A warhead swap (damage type +
  explosion) therefore leaves the surface-impact SOUND unchanged: the Dominator
  and the P/40-K already share `surface_impact_type` 4, so they thud identically.
  The audible change from a rechamber is the explosion the record now points at
  (`explosion_on_impact` 0→388), not the impact thud. If a round sounds wrong,
  check that explosion, not an impact-sound field that does not exist on the
  projectile record. Field names for unknown offsets are discovered the same way
  as type names (filediver json tags, see the reference).
- Two Lua files that share one scanner and differ only by `move_done` / `dmg_done` are one scan, not two. Read those flags before copying a function. The free-movement file's `patch_damage` also writes damage to 90, and `dmg_done` starts true so that writer never runs. The AP4 file writes `4/4/4/0` only. The combined mod uses the AP-only writer with both flags false.
- Do not write Dominator `max_ricochets` or the angle fields at offsets 136 and 180 to stop a tree or rock bounce. Those writes were logged in game and reverted. The measured results are in `references/install-and-format.md`.

Done when `/tmp/luavenv/bin/luaparser` exits 0 on the source file. There is no system `lua` or `node`.

## 5. Copy binary patches verbatim

The Lumberer Cremator flame is `0a4bd8a1833f11b2.patch_0` and `1577e917ad1f287d.patch_0`. They are byte-identical. Copy them. Do not rename, re-hash, or run them through the Lua packer. Keep the empty `.gpu_resources` and `.stream` sidecars beside them.

Done when `cmp` against the previous bytes reports nothing.

A full-bank swap goes stale when the game adds events. Before shipping another copy of an old `.bnk`, extract the current bank and diff HIRC type-4 event ids. If vanilla has an event the patch lacks, splice only the replaced WEM into the current bank and keep vanilla HIRC. List banks with filediver before scanning `data/bundles.*`. The FRV result and the Dixie container fields are in `references/install-and-format.md`. Done when every vanilla event id is still in the patched bank and only the intended WEM size changed.

## 6. Pack and replace the Arsenal copy

Container offsets are in `references/install-and-format.md`. The type at `0x50` says whether the payload is Lua. The filename `9ba626afa44a3aa3.patch_0` is a shared slot name, not a per-mod id.

The packer script is not in the mod folder. Reuse the script that wrote the current `Addon/*.patch_0`. If it is gone, rebuild the header from the three structs in `references/install-and-format.md`; the `length` field is `192 + len(payload)`, not rounded to 16. Before editing, write the packer and run it on the UNMODIFIED source first — it must reproduce the current `Addon/*.patch_0` byte-for-byte (`cmp` clean) — and only then edit and repack. That proves the header is right before your edits are in the way. Then check `len(patch) == 200 + len(lua)` and `patch[200:]` equals the source bytes. Offset `0x68` is MurmurHash64A of the addon-id string, seed 0. That matched `mods/maxigun/ap4_public` and `mods/dsh/dominator_bolt_pistol`. Offset `0xA0` is the Lua size plus 8. Leaving the old hash on a new addon id ships a header that names the wrong mod.

A single-option Lua mod is `manifest.json`, `thumbnail.png`, and `Addon/`. A multi-option mod uses named Include folders. Top-level options can all be on. Suboptions are pick-one. Put the suboption that is enabled now first, so a reimport does not flip it. A bank that is the whole mod has no Options key: `manifest.json`, `thumbnail.png`, and the `.patch_0` at the archive root. That is how SEAF-Chan is installed. Do not move that patch into `Addon/`.

A firing-sound bank is not part of the Lua Include, even when the filename is `9ba626afa44a3aa3.patch_0`. Classify it by the type at `0x50` and by whether byte 200 is Lua. The Dominator sound's type and folder are in `references/install-and-format.md`. Copy those bytes. Give it its own top-level option so it can be off while the gameplay option stays on. The manifest has no enabled field. On a mod that is already installed, set that option `enabled: true` in both `modsLibrary` and the selected profile. Copy `hd2a_data.json` aside first. Set the profile entry `enabled: true`, `deployed: false`, `changed: true` until Arsenal deploys.

Read that mod's `path` from `hd2a_data.json` immediately before the copy. Arsenal changes the `_AR` suffix; the suffix in the reference is not the live folder. Copying into the remembered path leaves the loaded copy on the old build. Replace that folder. Do not leave the old extract next to the new one.

Rebuild the zip under `~/clawd/hd2-mods/` in the same edit. That zip is what gets imported. From inside the source directory, `zip -X` the files so the archive root is `manifest.json`, `thumbnail.png`, and either the Include folders or the root `.patch_0`. No parent-directory entry. Check the namelist.

A zip with no `manifest.json` imports as a second mod. The label is the filename, the description is empty, and the uuid is new. Name, description, and Nexus identity are not in the patch. They are on both `modsLibrary` and `modsList[<selectedProfile>].mods` in `hd2a_data.json`. `manifest.json` needs `Version`, `Guid`, `Name`, `Description`, and `IconPath`. Use the installed uuid as `Guid` when the zip is that mod. Do not import it again to refresh the label.

HD2 Mod Manager (the Nexus one, teutinsa) stores the mod at `Mods\{manifest.Name}` and looks for `manifest.json` at the archive root. `Name` must be a legal Windows folder: no `<>:"/\|?*`. A colon fails the copy. `JAR-5 Dominator: Rechambered` is `JAR-5 Dominator - Rechambered`. Do not wrap the zip in a folder of that name. That hides `manifest.json` and the manager reports no manifest. An option `Name` is a label. The path is its `Include`. A name-only change does not bump `REVISION` and does not require a redeploy.

Nexus identity is `nexusData`: `modId`, `fileId`, `version`, `updateTimestamp`. A local mod stores `null`. To disassociate, copy `hd2a_data.json` aside and set `nexusData` to `null` on both records. The mod id in the folder name is the original download name, not the link. Both Dixie records stayed `null` on a re-read while `hd2arsenal` was still running. If the Nexus badge remains, the operator quits Arsenal and opens it again.

The operator still quits Arsenal, opens it again, and deploys before a game test. `deployed: false` is not a deployed mod.

A Dominator thumbnail change edits the shipped `thumbnail.png` (2048×2048). `branding/dominator_base_thumbnail.jpg` is 1024×1024 and is not that file. Do not generate a new image, and do not rebuild the release thumbnail from `dominator_render.png` or from the jpg. `view_image` rejects `/tmp`; copy a crop into the mod tree before reading it.

Done when the installed Lua payload equals the source, or the installed bank `cmp`s against the packed bank, each binary Include file `cmp`s clean, the zip namelist matches those files, and a Lua mod's packed `REVISION` is the shipped string.

## 7. Verify from the game log

Read the log path in the current source before looking for a file. Exosuit writes `$LOCALAPPDATA/CowboyBingus/Helldivers2/Logs/ExosuitAmmo.log`. Under Proton that prefix is `~/.steam/steam/steamapps/compatdata/553850`.

Mod Lag Watchdog is installed (`mods/patpatpatrick/mod_lag_finder`). Absence from its cost list means the mod is under the display threshold. A stale name is the watchdog reading an empty Proton `hd2a_data.json`. Arsenal's list is `~/.config/hd2arsenal/hd2a_data.json`. A steady ms/s line **labelled `modname (+ all below it)`** is an attribution bug, not the mod's cost. The watchdog follows the update chain through direct upvalues (`previous_update`, `original_update`); a hook that stashes the previous update in a table field and calls it through a closure (`BUS.base = update; pcall(BUS.base, ...)`) cannot be followed, so the watchdog bills "the mod + everything below it" — the game's own ~0.7 ms/frame update loop — to the mod. Realistic Weapons did this with a shared `OCLAW_UPDATE_BUS` dispatcher and showed a steady ~58 ms/s that never matched its own idle log; the hitch lines proved it (`realistic_weapons (+ all below it) 0`, the 57 ms "not explained (outside update)"). Fix: replace the dispatcher with a direct upvalue hook — `local original_update = update; update = function(...) pcall(tick) return original_update(...) end` — which both lets the watchdog follow the chain and removes the `pcall(base, ...)` JIT barrier on the game's hot path. Check whether the shared bus has other users first (grep the install dir for its global name). A per-frame `os.time()` / `os.date()` call in `tick()` is a real but SMALL steady cost (microseconds, ~3 ms/s, not tens) — convert it to frame counts (`state.frame + 300/600` ≈ 5/10 s) after fixing the hook. `os.clock()` inside a search-phase budget loop is fine — that loop does not run in steady state. Realistic Weapons' hook is `mods/recon/realistic_weapons`. A line that still says `realistic_mg206` is the checkbox id from before the hook moved.

`Helldivers 2/data/9ba626afa44a3aa3.patch_0` is not the Lua bundle. That name in the game tree was a binary patch. Lua loads from the Arsenal folder. Binary patches show up in `data/` under indexed names (`9ba626afa44a3aa3.patch_118`). Match those by size, not by the shared filename.

Done when the log line shows the shipped `REVISION` and, for an experiment that kept that revision, the test token. The in-game numbers must match the manifest, or the reply quotes the log line that failed. `deployed: false` means that check has not happened yet.

## 8. Value-scan patching (health / armor)

Some tables are found by searching memory for a known VALUE, not the LDLD type
hash — the record array address cannot be derived from the header. The exosuit
HealthComponentData is the example: scan for health values 1800 (hull), 550
(legs), 800 (arms), then walk back to the record. Offsets in the reference.

A value-scan is dangerous. A loose match ("main health 1800", or "field at +280
== 4") matches unrelated data and corrupts the game (black screen / silent
close). Require the FULL signature before writing — for the mech body: main
health 1800 AND default armor 4 AND at least one zone with a known health
(400/550/10) — then write only the confirmed zones. Read back and roll back on
every write, exactly like the type-hash path. Never match a record by a value
this mod already wrote.

A value-scan is also the launch-time cost: it walks the WHOLE process (~4 GB)
for the value, every chunk, and retries the whole pass when a separate table is
the straggler. Bound it two ways. **Stall the value-scan once its table is
passed** — the mech table is ~11 MB contiguous, so once a write has landed and N
chunks (256 ≈ 32 MB) pass with no new record, stop scanning for that value.
**Early-exit the whole scan once every subsystem reports applied** (magazine,
projtype, armor, rockets each `> 0`), instead of finishing the remaining GB and
re-running up to `MAX_ATTEMPTS` full passes. The stall/exit only stop the
search; the full-signature guard above still decides what is written.

The settle floor is the game's load order, not scan speed. The straggler tables
(WeaponMagazine/ProjectileWeapon) can be absent from memory until ~60 s after
launch — the mod retries until they appear, and no amount of scanning finds a
table that is not resident yet. When the spike is already fixed, do NOT keep
lowering the frame budget to chase a shorter settle: a smaller per-frame budget
only trades the spike back for more wall-clock duration and risks `MAX_ATTEMPTS`
expiring before the straggler loads. The remaining settle is the game loading,
not mod overhead.

## 9. Pitfalls that each cost a game test

- **Local declared after use.** A `local x` placed below the function that uses
  `x` makes it a nil global inside that function — "attempt to compare nil with
  number" at the first comparison. Declare locals before the functions that
  reference them. `luaparser` does not catch this.
- **Arsenal overwrites direct edits while running.** Editing a mod folder and
  `hd2a_data.json` while `hd2arsenal` is open gets reverted on its next save.
  Quit Arsenal, edit, reopen, deploy — or import a rebuilt zip instead.
- **Resource hash vs entity hash.** A game object's network descriptor
  `resource` (u64) can be the same value as the table's entity key for a given
  weapon, but never assume it — verify against SHODAN's WEAPONS catalog
  (`src/stat_editor.lua`, third column) or HD2Runtime's capability catalogs.
  The Dominator's entity key AND resource are both `0x80F1A156D9FA1E36`.
  `0xB6AFF2195568767F` is the **R-36 Eruptor**, not the Dominator — an earlier
  Dominator build pointed `DOMINATOR_ENTITY` at the Eruptor and silently
  installed the ammo wheel + recoil on the wrong weapon. Compare the right one.
- **"Raise to value" is not "+1".** Raising AV3->AV4 leaves AV4 untouched; "+1
  to all" also bumps AV4->AV5 and AV0->AV1. The "+1" version writes more fields
  and is less selective, so it needs the full signature guard; a plain raise
  was safe because it only wrote exact value 3 with exact health 550/800.
- **A maintain loop that re-asserts the whole record fights live editors on
  shared fields.** `maintain()` compared the full cached record and rewrote every
  field on any mismatch. SHODAN's projectile-speed editor writes the speed field;
  our maintain saw the record "changed" and rewrote speed to 380; SHODAN rewrote
  its value; forever — cross-process writes every ~5 s (lag spikes). This is the
  runtime twin of section 2's install-time overlap. Make maintain field-selective:
  on mismatch, diff only the fields this mod owns (warhead damage type + explosion
  on impact/expire, arming) and leave shared ballistics (speed, range) to the
  other editor, or add a config key to hand the field over — the shipped syntax
  is `speed=off` (or `speed=N` for a custom velocity), not `speed=-1`, which
  silently leaves the default: a bare `-1` passes neither the `"off"` string
  branch nor the `n > 0` numeric branch. The recognition cue is a user report of
  "it constantly goes back and forth with it."
- **A per-frame/per-second HUD poll hitches the game.** A badge that must appear
  only while a weapon is equipped still needs a poll, but the expensive walk
  (`objects_owned_by` + `game_object_is_type` per owned object + two hash-table
  walks, dozens of ReadProcessMemory calls) must not run every 60 frames. That
  caused audio buffer underruns ("sound kept cutting out") and dropped hit
  registration ("shot center mass, bot not hit") — both gone on reverting to the
  version without the HUD. Throttle the equipped-check to ~5 s (match
  `VERIFY_EVERY`), cache the game.dll base and the player avatar (invalidate on
  `net` pointer change / failed lookup), and only redraw when the equipped state
  actually changed. The Dominator badge was removed in v3.1; the ammo wheel
  (section 11) that replaced it is the native programmable-ammo selector, not an
  overlay — for a state a native UI already renders, deleting the overlay beats
  throttling it.

## 10. HD2Runtime (Skyeshade SDK) and SHODAN ecosystem

The typed-SDK alternative to hand-rolled scanners. `mods/skyeshade/hd2runtime`
is a read-only shared runtime (Bingus v15+/API 1): it owns memory access, page
protection, timers and resolution; dependent mods declare declarative
transactions instead of scanning. API shape:

```lua
local hd2 = require('mods/skyeshade/hd2runtime')
local p = hd2.weapon('AR-23 Liberator'):attack('primary'):projectile()
return hd2.ensure({ transaction = { id='x', target=p, allow_shared=true,
    changes = { { field=hd2.fields.damage.player_standard_damage, expect=90, value=120 } } }})
```

- `hd2.fields.*`, `hd2.enums`, `hd2.resources` are the discoverable constants;
  `allow_shared` + `expect`/`value` is the framework-level solution to the
  write-conflict problem section 9 documents — the runtime detects shared
  writers and refuses to fight.
- The runtime blob is 16 MB of 93 embedded Lua chunks (capability catalogs), not
  a single readable source. It cross-confirms our hashes: `0xBD4042C2`
  (ProjectileSettings) x1735, `0xE0A72CF0` (DamageSettings) x1984,
  `0xFB8D88A3` (WeaponMagazine) x29, Dominator resource `0x80F1A156` x12.
- **HD2Runtime has NO HealthComponentData** (`0xB3915DE3` = 0 occurrences).
  The exosuit health/armor value-scan technique (section 8) fills a gap the
  ecosystem SDK does not cover.
- **Presentation labels (the armory AP/trait display) are writable, not native
  code.** `hd2.fields.presentation.armor_penetration` (allowed `none`/`light`/
  `medium`/`heavy`/`light_anti_tank`/`anti_tank`) and `presentation.traits` both
  back onto `LoadoutEntryComponentData` at +12, storage `armor_penetration_label`
  / `trait_set`, width 20 (five trait tags); `SLOT_SETS` maps `trait_set=5`,
  `armor_penetration_label=5`. This is the fix for a mod whose warhead/damage is
  patched but whose armory label never moves. Both are flagged
  `allow_unverified_effect` ("an edited label appearing in the menus has not been
  gameplay-tested") and "menus build their labels when they open" — expect the
  change on reopen, not live. The dlsum type hash of `LoadoutEntryComponentData`
  is not yet extracted; HD2Runtime names it by field, so the API route needs no
  hash.
- SHODAN Stat Editor is a dependent of HD2Runtime (its weapon/stratagem
  catalogs credit it). Its field layout is authoritative for projectile/damage
  offsets: projectile stride 272 (velocity +32, drag +40, gravity +44, pen
  slowdown +64, damage type +60, explosion +144/+156); damage stride 76 (damage
  +4, durable +8, ap_direct +12, ap_slight +16, ap_large +20, ap_extreme +24,
  demolition +28, stagger +32, push +36). Matches our verified layouts.
- The separate `SHODAN Stat Editor简体中文` download is a translation-only
  addon (`mods/shodan/stat_editor_zh_cn`); it hooks `Gui.text`/`text_extents`
  and writes no stat. The actual editor is `SHODAN-Stat-Editor` on GitHub.

## 11. Programmable ammo (weapon-wheel alternate round) — point at a real round

The Autocannon's flak/APHET selector is the game's **programmable-ammo**
function. The three writes that build it are below. The v3.0 build tried to
hand-roll a spare projectile record and shipped a game-breaking bug; the fix is
to never clone — point the function slot at a real, referenced round instead.

1. **Spare projectile — DO NOT clone, do not hunt for "dormant".** The v3.0
   build "found" a spare by looking for a record referenced by no weapon
   (ProjectileWeapon +0/+576) that looked dormant (zero name hash or zero
   speed). **That heuristic is wrong.** Enemy projectiles are referenced OUTSIDE
   ProjectileWeapon (enemy attacks, shrapnel, beams, orbitals) and legitimately
   carry `name==0`, so they look dormant. The Bile Titan's spit is ProjectileType
   79 — zero name — and the clone overwrote it with the Dominator's explosive
   round ("bile titans spitting bolt rounds," titans ragdolling and instakilling
   players). **You also cannot append a new record:** the projectile table is a
   memory-mapped `dl_bin` the game deserializes at load into a fixed 350-record
   registry, so a 351st record or a brand-new type id is never referenced by the
   ammo-swap slot. `name==0` / `speed==0` is NOT evidence of "dormant."

   **The safe alternate is a round that already exists and is already referenced.**
   For the Dominator, that is the P/40-K's own projectile — the exact round the
   rechamber copies its warhead FROM, which natively fires the explosive warhead
   at 0.1 m arming. Point the function slot straight at it: SAFE (Dominator, 12 m
   arming, 380 m/s) vs UNSAFE (P/40-K's stock round, 0.1 m arming, 350 m/s). Zero
   projectile records are cloned or overwritten, so nothing outside the Dominator's
   own record is written (the only other write is `mode_label`/`mode_icon` on the
   source round, presentation-only). If both rounds must share an exact stat (e.g.
   both at 380 m/s), that is the unsolved clone problem again — do not solve it by
   overwriting an enemy projectile.
2. **Function-projectile slot.** `ProjectileWeaponComponentData` (`0x45171B68`)
   +576 = the alternate round's type. This is the round fired when the alternate
   is chosen.
3. **Selector binding.** `WeaponDataComponentData` (`0x88E4DBB1`, stride 1232)
   `function_info` at +184 (left) / +188 (right), u32, = 8 (`programmable_ammo`).
   Bind whichever input is currently `none` — the Dominator's left is free, its
   right is already `fire_mode` (single/burst).
4. **Ammo icons + labels** (the same regardless of where the alternate round
   comes from). The wheel renders each round's own `mode_label`
   (ProjectileSettings +12, u32 localisation id) and `mode_icon` (+16, u64 icon
   resource) — presentation fields on the projectile record itself, not on the
   weapon tables. Plain projectiles leave both zero, so a wheel with no icon/label
   writes shows blank entries for both options. The label and the icon are
   independent — set them separately, and you can swap which icon goes on which
   round freely.

   - **Label is a RAW localisation string id**, not limited to the
     `modePresentation` catalog's offered labels. Any string in the game's
     `.strings` database resolves. To make the wheel read a specific word, find
     that word's id: extract the strings and read its `Key`:
     ```bash
     filediver --gamedir "$STEAM/.../Helldivers 2" -o /tmp/str \
       --types strings --strings-language "English (US)" -i "*"
     # then grep the *.strings.json for "Value": "<word>"; its "Key" is the u32 id.
     ```
     Native standalone "Safe" = 782820489, "Unsafe" = 622682013 (title-case;
     HD2 has no all-caps SAFE/UNSAFE string). A written id localizes to the
     player's language. Do not guess a string id by hashing the word yourself —
     read the `Key` from the extracted database.
   - **Icon** is a native weapon-function icon resource from
     `modePresentation.modeIcons` → `resource` (an `0x` hex). The u64 is
     little-endian — encode with `entity_le8(hi,lo)` (hi/lo halves of the hex)
     and it matches the runtime's `resource_bytes`. Icon resources offered:
     `ammo_he` `0x081DCAF42DC2ECBB`, `ammo_aphet` `0xF8EDE5F922C05CB6`,
     `ammo_slug` `0xB7812FA260A5F64F`, etc.

   The wheel shipped SAFE = "Safe" (782820489) + `ammo_aphet` icon and
   UNSAFE = "Unsafe" (622682013) + `ammo_he` icon. The string-id and
   icon-resource values above are current as of the v3.1 source-round wheel;
   read them back from the extracted database rather than trusting memory.

Enum (u32): none=0, zeroing=1, rate_of_fire=2, magazine=4, fire_mode=5,
muzzle_velocity=7, programmable_ammo=8.

**Wheel shows but both icons are blank** — the `mode_label` / `mode_icon` fields
on the projectile records are zero; write the native values (step 4), not a
rendering fix. This is distinct from "wheel does not show at all" below.

Caveat: HD2Runtime marks giving a weapon a function projectile as
`allow_unverified_effect` — "the ProgrammableAmmo fire path is proven, but giving
this weapon a function projectile has not been gameplay-tested." Log every write
and confirm the wheel shows the alternate before shipping.

**When the wheel does not show, first prove the writes landed on the right
weapon** — do not blame rendering or selector conflicts until you have. The
log's own `ammo: ProjectileWeapon record at 0x… (projtype N, function M)` line
names the weapon that actually got the selector. `projtype` is the weapon's
round: the Dominator is `projtype 177`; `projtype 40` is the R-36 Eruptor and
proves the keyed-table lookup resolved the WRONG entity hash (section 9's
resource-vs-entity pitfall) — every write (wheel, source-round function slot,
and recoil) silently landed on a different weapon while the log still printed
"selector bound". Check that `projtype` against the intended weapon before touching the
wheel logic. A weapon that natively has a fire-mode selector and no ammo selector
*may* still not render the option even with a correct hash, but that is the
second hypothesis, not the first.
