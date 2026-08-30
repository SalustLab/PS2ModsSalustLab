# Klonoa 2: Lunatea's Veil (USA) — Hard Mode **v1.3** (2026-08-30)

Patch for `SLUS_201.51` (NTSC-U, PCSX2 CRC `2F56CBC9`). Only the executable changes (230 words); no asset is touched. v1.0 items were tested in-game on PCSX2; the speed-up shipped in v1.1 (1.25×) was play-tested and bumped to **1.5×** in v1.2; v1.3 fixes the Biskarsh chase (the tank was being sped up — it is an enemy of class `0x80B`, not a level script). The rest of the exclusion checklist at the end is still open.

## What changes

| Item | Original | Hard Mode |
|---|---|---|
| Max HP | 3 | **2** (HUD draws 2 hearts) |
| Healing | small heart +1, Double +2, cap 3 | same items, cap **2** |
| Small heart | always heals | **heals only 20 % of the time** (rolled with the game's own RNG on pickup; the item still appears normally — Dream Stones and 100 % runs are untouched). If it fails at 1 HP, an in-game warning sound plays **instead of** the normal pickup sound (at full HP, original behaviour). |
| Double heart | always heals | unchanged — guaranteed heal (rare, designer-placed item) |
| Boss "comeback" (1UP capsules when `lives < 3`, up to 2× per session) | active | **removed** |
| Starting lives | 3 | **2** (0 is still playable → 3 attempts; applies to a **new save** — an existing save keeps its stored lives) |
| Max lives | 99 | **4** |
| **Enemies and bosses** (v1.1/v1.2) | 60 ticks/s | **1.5× faster in every sense** — AI timers, movement, attacks, projectiles, hitboxes *and* animation, all together (the object gets 3 update ticks and 3 animation steps every 2 frames; v1.1 shipped 1.25×). Klonoa, items, moving platforms/traps and the camera stay at normal speed. **Not affected**: an enemy while Klonoa holds it (and its held-timer), a thrown enemy, Polonte the Hatchling, King of Sorrow phase 1, the Biskarsh tank in the Volk Inferno chase (enemy class `0x80B`, fixed in v1.3; the regular enemies of that area are still faster), scripted mini-bosses (Pop-Up Tat) and the boss "comeback" capsules. |
| Damage per hit, knockback, i-frames, 1UP at 100/200 stones | — | unchanged |

## How to apply

**ISO (xdelta):** `Klonoa2_HardMode.xdelta` applies to the original `Klonoa 2 - Lunatea's Veil (USA).iso`
(SHA-256 of source and result are in `CHECKSUMS.txt`).

- GUI: [Delta Patcher](https://github.com/marco-calautti/DeltaPatcher) → Original ISO / XDelta patch / Output.
- CLI: `xdelta3 -d -s "Klonoa 2 - Lunatea's Veil (USA).iso" Klonoa2_HardMode.xdelta "Klonoa 2 [Hard Mode].iso"`

**PCSX2 (no ISO rebuild):** copy `SLUS-20151_2F56CBC9.pnach` to `Documents\PCSX2\cheats\` and enable
*Enable Cheats* in the game's properties. Same changes, applied in RAM on top of the original ISO.

## Tuning the small-heart heal chance

In the pnach, the line `patch=1,EE,0032DF34,word,24010005` is `addiu at,zero,5` → heals when `rng % 5 == 0` (20 %).
Change the last byte: `24010004` = 25 %, `2401000A` = 10 %, `24010002` = 50 %, `24010001` = always heals (disables the effect).

## Changing the "heart failed" sound

The id is a 64-bit value built by 6 words in the code cave (see the pnach comments: `lui/ori/dsll/ori/dsll/ori`). Default `000C80C85CA00400`. Candidates (all in-game SFX, picked by usage context — menu/HUD): `004B04B05CA00401`, `004B04B05CA00402`; or `004B04B01CA00303` (blue Dream Stone sound). The clean way is to change `FAIL_SFX` in `build_hardmode.py` and rerun; for quick A/B, edit the four 16-bit halves in words 1, 2, 4 and 6 of the block in the pnach.

## Tuning the enemy/boss speed (v1.1+)

All knobs are plain words in the pnach (comments mark them) or constants at the top of `build_hardmode.py`:

- **Speed factor** — `TICK_MASK` (two identical `andi at,at,N` words, marked `KNOB TICK_MASK`; edit both): the extra tick happens when `frame & N == 0`, so speed = `1 + 1/(N+1)`. `N=1` → **1.5×** (v1.2 default), `3` → 1.25× (v1.1), `2` → 1.33×, `7` → 1.125×, `0` → 2×, `FFFF` → effectively off.
- **Which enemies** — `MASK_ENEMY` (word at `0x266A38`): bit *b* = kind-3 subtype `0x800+b`. Default `0003F17F` = classes `0x800–0x806, 0x808, 0x80C–0x811` (the ones that use the enemy hit/capture handler). Class `0x80B` is off since v1.3: the **Biskarsh chase tank** (Volk Inferno, level id `0x0801`) is a class-`0x80B` enemy — verified live with `k2probe.py` — and it is unplayable at 1.5×. `FLAG_200A` (`0x266A3C`) adds subtype `0x200A`.
- **Which bosses** — `MASK_BOSS` (two words at `0x266A40/0x266A44`, bit = kind-7 subtype 0–63). Default whitelists the boss actors and their children: `0x01 0x03 0x04 0x07 0x13 0x1A 0x1F 0x23 0x24 0x29`. Not whitelisted on purpose: `0x0A` (Polonte), `0x10` (King of Sorrow phase 1), `0x14` (comeback capsules), every level/cutscene script (`0x22` etc.).
- **Excluded levels** — list of level ids (`g_game+4` = RAM `0x366454`) from `0x266A48`, terminated by `FFFFFFFF`. Default `{0x1B00}` (King of Sorrow phase 1 — it shares child subtypes `0x1A/0x1F` with other bosses).
- **Excluded species** — list of enemy species ids (`obj->0x18->0xE`) from `0x266A50`, terminated by `FFFFFFFF`. Empty by default; use it if a specific kind-3 enemy must stay at normal speed.

To find a level id or an object's kind/subtype in PCSX2: Debug → Memory, read `0x366454` (level id), or inspect the 128-slot object array (base pointer at `0x36EB00`, stride `0xF0`; `+0x1C` kind, `+0x1E` subtype). The clean way to change anything is editing the constants and rerunning `build_hardmode.py`.

## Addresses (EE, for auditing)

- `g_game = 0x00366450`; HP `+0x400` (`0x366850`), lives `+0x408` (`0x366858`).
- Initial HP: `0x1A6B8C`, `0x1A6C78`, `0x1B4C64`, `0x194DA8`; clamps `0x194D9C`, `0x1D410C`, `0x1D4174`, `0x15A1E4`; heals `0x1D4118`, `0x1D4180`, `0x15A1E8`; HUD `0x192440`.
- Comeback: `0x159E1C`, `0x159F58`, `0x15A048` (`slti v0,v0,2` → `move v0,zero`).
- Lives: start `0x17CD84` (new save-block init — the real source; `0x199524` is only the boot init); cap `0x1956B0/0x1956BC`, `0x15A25C/0x15A260`.
- Heal roll: hook `0x1D4104` (the small-heart case's `addiu v0,v0,1` in the pickup handler → `jal`) → code cave `0x32DF14–0x32DFF0` (zeroed padding at the end of `.text`): saves regs, calls the game RNG (`0x1227C8`), `divu` by N, heals only if the remainder is 0. Double heart (`0x1D416C`) untouched.
- Heart pickup sound: the normal sound (`004B04B01CA00301`) is played by the *effect-queue processor* (`0x1D39F8`; case `0x815` at `0x1D3A78` → common block `0x1D3CD8` → `0x1F9D28`), not by the pickup handler (which only queues the effect via `0x1D3858`). The `b` at `0x1D3A90` becomes a stub that, if the RAM flag `0x32DFFC` (set by the cave on failure) is on, consumes it, **zeroes the queue entry** (the queue is rescanned every frame and only `0x1F9D28` disarms an entry after playing — without zeroing, the ding would play one frame later) and skips the sound. Double (`0x814`, id `…0300`) has its own path and is unaffected.

- Speed-up (v1.1): the game is fixed-step (60 ticks/s, no delta time). One loop updates every object (`0x1A5BE8`: 128 slots × `0xF0`, `jalr [0x375C00 + kind*4](obj)` at `0x1A5C54`) and three render passes call each object's draw callbacks (`0x18DE88/0x18DEF8/0x18DF68`, `jalr obj->4/8/0xC` at `0x18DECC/0x18DF3C/0x18DFAC`); the draw callback is what advances animations (`0x20F448 → 0x20AC00 → 0x20DF90`, `frame += speed` at `0x20E14C/0x20E180/0x20E1E0`). The four hooks `jal` into a cave placed in a dead function (`0x266A30`, one of 8 identical never-referenced copies): `QUALIFIES(obj)` = kind 3 with subtype in `MASK_ENEMY`/`0x200A`, or kind 7 with subtype in `MASK_BOSS`, minus excluded levels/species. Update hook: after the normal tick, if `frame & TICK_MASK == 0` and the object still qualifies (re-checked *after* the tick — a captured enemy has already become kind 2, a dead one kind 0), run the update once more with the RAM flag `0x266A30` set. Render hook: same test, sets flag `0x266A34` around the draw callback; the three animation sites add `speed` a second time while that flag is on. Held enemies are kind 2 (`0x1E1000` converts the object on capture), thrown ones kind 1 — never qualified. Bosses are kind-7 state machines created per level by the hub `0x1228E8` (`0x1500→0x23`, `0x1600→0x01`, `0x1700→0x03`, `0x1800→0x0A`, `0x1A00→0x04`, `0x1B00→0x10`, `0x1B01→0x29`, `0x1B02→0x13`, `0x1102→0x22` chase script).

## In-game validation checklist for v1.1

1. Regular stage: enemies visibly faster (walk/attack/animation in sync); Klonoa, platforms, items, camera normal. *(v1.1 result: OK per play-test.)*
2. Capture an enemy: normal speed while held (and its fuse, if any), normal throw.
3. Whitelisted boss (e.g. the first one, level `0x1500`): attacks, projectiles and animation faster, no "waiting" at the end of animations.
4. Polonte and King of Sorrow phase 1 at normal speed; King of Sorrow later phases faster. If a level/boss mapping is off, fix `MASK_BOSS`/`EXCL_LEVELS` (read `0x366454` in that fight). *(Biskarsh chase: v1.2 play-test found the tank sped up → class `0x80B` removed from `MASK_ENEMY` in v1.3; re-check that it is normal now and that the area's regular enemies still are faster.)*
5. Pop-Up Tat at normal speed; if not, add its species/level to the exclusion lists.
6. Classes `0x807`/`0x80A` (spawner-like, not in `MASK_ENEMY`): if some enemy type is *not* faster, add its bit.
7. ~30 min without crash or soft-lock; v1.0 behaviour (HP 2, 20 % hearts, lives 2/4, no comeback) unchanged.

Generated by `build_hardmode.py` (Python + capstone); every site is verified by disassembly before/after.
