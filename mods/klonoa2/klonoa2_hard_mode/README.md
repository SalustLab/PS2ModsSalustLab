# Klonoa 2: Lunatea's Veil (USA) — Hard Mode **v1.5** (2026-08-30)

Patch for `SLUS_201.51` (NTSC-U, PCSX2 CRC `2F56CBC9`). Only the executable changes (319 words); no asset is touched. Every mechanic below was play-tested in-game on PCSX2 across v1.0–v1.5.

## What changes

| Item | Original | Hard Mode |
|---|---|---|
| Max HP | 3 | **2** (HUD draws 2 hearts) |
| Healing | small heart +1, Double +2, cap 3 | same items, cap **2** |
| Small heart | always heals | **heals only 20 % of the time** (game's own RNG, rolled on pickup). If it fails below full HP, a warning cue plays **instead of** the pickup ding. |
| Double heart | always heals | unchanged — guaranteed heal (rare, designer-placed item) |
| Starting lives | 3 | **2** (0 is still playable → 3 attempts; applies to a **new save**) |
| Max lives | 99 | **4** |
| 1UP item pickup (v1.5) | always +1 life | **50 % chance** (RNG on pickup). On a failed roll — or when already at 4 lives — the warning cue plays instead of the jingle and no life is gained. 1UPs from 100/200 stones are untouched. |
| Boss comeback 1UPs (v1.0) | capsules when `lives < 3` | **removed** |
| Boss item drops (v1.5) | bosses drop item capsules on a hit-counter + RNG (health etc.; KoS drops many) | **removed** (all 7 boss drop call sites disabled) |
| **Enemy & boss speed** (v1.1–v1.5) | 60 ticks/s | Per-group/per-fight rates via extra update+animation ticks — AI timers, movement, attacks, projectiles, hitboxes *and* animation together. **General: 1.5×**. **Leptio (+ flowers): 1.25×**. **Folgaran (body, claws, spinning spikes, and the Moos in the arena): 2.75×**. Klonoa, items, platforms/traps and the camera stay 1×. **Not sped**: held/thrown enemies, Polonte, the Biskarsh chase tank (class `0x80B`), Pop-Up Tat, and the three mini-boss capture arenas (fully vanilla zones). |
| Damage per hit, knockback, i-frames | — | unchanged |

## How to apply

**ISO (xdelta):** `Klonoa2_HardMode.xdelta` applies to the original `Klonoa 2 - Lunatea's Veil (USA).iso` (SHA-256 of source and result in `CHECKSUMS.txt`).

- GUI: [Delta Patcher](https://github.com/marco-calautti/DeltaPatcher) → Original ISO / XDelta patch / Output.
- CLI: `xdelta3 -d -s "Klonoa 2 - Lunatea's Veil (USA).iso" Klonoa2_HardMode.xdelta "Klonoa 2 [Hard Mode].iso"`

**PCSX2 (no ISO rebuild):** copy `SLUS-20151_2F56CBC9.pnach` to PCSX2's `cheats\` folder and enable *Enable Cheats* in the game's properties. Note: PCSX2 names cheat files by ELF CRC — that pnach matches the **original** ISO; a patched ISO has a different CRC (`build_hardmode.py` emits a matching pnach per CRC if you need cheats on top of the patched ISO).

## Knobs (all are plain pnach words; comments in the pnach mark each one)

- **Heart heal chance** — `addiu at,zero,N` at `0032DF34`: heals when `rng % N == 0`. `5` = 20 % (default), `4` = 25 %, `2` = 50 %, `1` = always.
- **1UP gain chance** — `KNOB ROLL_1UP` (`andi v0,v0,N`): gains when `rng & N == 0`. `1` = 50 % (default), `0` = always (vanilla), `3` = 25 %.
- **Warning cue sound** — 64-bit SFX id built by two `lui/ori/dsll/ori/dsll/ori` blocks (heart cave + 1UP cave). Default `000C80C85CA00400`; change `FAIL_SFX` in `build_hardmode.py` and rebuild.
- **Speed rates** — 4-byte tick **patterns** (`KNOB PAT_FAST`, `KNOB PAT_SLOW`, and per-level `KNOB BOOST[n]` level/pattern pairs). Each byte = extra update+animation ticks on frames where `frame&3` equals the byte index; speed = `1 + sum(bytes)/4`, so anything from 1.0× to 3.0× in 0.25 steps: `00000001`=1.25×, `00010001`=1.5×, `00010101`=1.75×, `01010101`=2×, `01020102`=2.5×, `01020202`=2.75×, `02020202`=3×.
- **Which enemies** — `KNOB MASK_ENEMY` (bit *b* = kind-3 class `0x800+b`; default `0003F17F`, class `0x80B` off = the Biskarsh chase tank). `KNOB FLAG_200A` adds class `0x200A`.
- **Which bosses/actors** — `KNOB MASK_BOSS` lo/hi (bit = kind-7 subtype). Whitelisted: bosses + their children incl. Folgaran spikes (`0x09`), Biskarsh missiles (`0x06`), KoS iron balls (`0x11`) and green orbs (`0x12`). Excluded on purpose: Polonte (`0x0A`), comeback capsules (`0x14`), Pop-Up Tat (`0x16/0x17/0x25` — speeding it desyncs its bomb-throw pipeline and floods the object array), every level/cutscene script.
- **Slow group** — `KNOB MASK_BOSS_SLOW` lo/hi: whitelisted subs that use `PAT_SLOW` (default Leptio `0x01`+`0x02`).
- **Boost levels** — `KNOB BOOST[n]` pairs: in that level id, every qualified object (enemies included) uses that pattern (default: `0x1500` Folgaran → 2.75×).
- **Excluded levels / species** — `KNOB EXCL_LEVELS` / `KNOB EXCL_SPECIES` lists (end `FFFFFFFF`). Default levels: `0x0401, 0x040A, 0x0605` = the three scripted mini-boss capture arenas (fully vanilla).

To inspect the game live (PCSX2 → Settings → Advanced → Enable PINE): `python k2probe.py status|objects|watch` shows the level id, every object's kind/subtype/state and the speed the mod applies to it.

## Addresses (EE, for auditing)

- `g_game = 0x00366450`; HP `+0x400`, lives `+0x408`, level id `+0x4`, frame counter `+0x454`.
- HP/lives/comeback/heart-roll sites: unchanged from v1.0 (see git history) — heal-roll cave at `0x32DF14–0x32DFF0`, fail-flag `0x32DFFC`, sound-skip stub hooked at `0x1D3A90`.
- **1UP roll (v1.5)**: hook `0x1D41B0` (`j` into the cave) in the 1UP item handler `0x1D418C` (pickup jump table `0x3EDDF0` idx 5/9). The cave checks lives≥4 → cue; else RNG roll → cue or original jingle (`004B04B01CA00306`) + gp flag + `lives++`; exits at `0x1D41F0`. Stone-count 1UPs use a different path and are untouched.
- **Boss capsule drops (v1.5)**: spawners `0x159900/0x159A60/0x159B60` (capsules = kind-7 sub `0x14`); the 7 boss call sites NOP'd: `0x134944, 0x137E88` (Folgaran), `0x13E32C` (Leptio), `0x14FF10, 0x153E54` (KoS), `0x14B2B4` (Leorina), `0x1449D4` (Biskarsh). Delay slots still reset the hit counters.
- **Speed engine**: object update loop `0x1A5BE8` (hook `0x1A5C54`), render passes `0x18DE88/0x18DEF8/0x18DF68` (hooks `0x18DECC/0x18DF3C/0x18DFAC`), animation advance `0x20DF90` (hooks `0x20E14C/0x20E180/0x20E1E0`). Cave in a dead function at `0x266A30` (one of 8 identical never-referenced copies). `QUALIFIES(obj)` returns the frame's extra-tick count from the group pattern; the update hook runs that many extra ticks **but stops if the object's state (`+0x24`) changed this frame** (transition guard — a freshly-entered state polls its animation only after the render, like vanilla; prevents premature-fire bugs such as Folgaran's arms losing their draw callback on respawn); the render hook stores the count for the animation sites (up to 2 extra `add.s` per frame).
- **Anti-softlock gates**: three boss state handlers gated transitions on an *exact* anim-frame equality, which extra anim steps can skip (Leptio froze in T-pose). Patched `c.eq.s` → `c.le.s` ("reached") at `0x13C648`, `0x13CFB0` (handlers shared by Leptio and Folgaran's claws) and `0x1612BC` (Tat). Vanilla timing is identical; SFX-only `==` cues elsewhere were left alone.
- Boss actors are kind-7 state machines created per level by the hub `0x1228E8`: `0x1500→0x23` Folgaran, `0x1600→0x01` Leptio, `0x1700→0x03` Biskarsh, `0x1800→0x0A` Polonte, `0x1A00→0x04` Leorina, `0x1B00→0x10` / `0x1B02→0x13` / `0x1B01→0x29` King of Sorrow. The Biskarsh chase (level `0x0801`) tank is a kind-3 class-`0x80B` enemy.

Generated by `build_hardmode.py` (Python + capstone); every patched site is verified by disassembly before/after, with original-word asserts on all hooks.
