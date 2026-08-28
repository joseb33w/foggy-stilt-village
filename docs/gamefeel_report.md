# Game-Feel / Mobile-UX report — "Heronwade" CONTINUE build (Mire Dragon encounter)

**VERDICT: PASS — 0 P0, but 3 ❗P1 must-fix feel defects in the new boss encounter.**

Build: `cloud-hdro8hilebm5ifgmt2hy` · Scope: session delta (The Mire Dragon @ cell [-7,10], "The Dragon's Roost") + mobile HUD/touch regression.
Method: drove a local offline copy of the export (multiplayer off, start_cell at the roost — copies only, repo/out untouched) in software-GL Chromium at **portrait 400×860 and landscape 860×400**, using **real CDP touch events** (not mouse), across **5 full boots** (~15 min of play). Evidence frames in `/tmp/feel/*.png` (p400/f400/l860/b400/k400/c400 series). Note: software GL dilates game-time ~2–3× vs wall clock; all timings below account for that.

---

## ❗ P1 — The boss disengages / never engages: its hover stand-off ring is OUTSIDE its own attack range (spawn-time dice roll)

**Symptom (observed, 5/5 boots):** the fight never sustains. In 2 of 5 boots the dragon **never attacked once** in 200+ s while the player stood (and attacked) in the middle of its roost — it hovers ~8 m off as a passive statue (screenshots `b400_bend`, `k400_b99`: full player HP, dragon only a dot on the minimap for 3+ minutes). In a third boot it took 11 sweep-the-roost movement cycles to force a single hit (`c400` log: `dragon_aggro` only at t=247s). In the boots where it did engage immediately (spawned close), it landed ~8 hits then drifted back out and the player passively regenned to full (`f400_calm1`).

**Root cause (verified in code):** `enemy.gd` — flyers hold a stand-off slot at `max(SWIM_KEEP, surround_radius) * _air_keep` (line ~321), with `surround_radius = attack_range * 0.85` (line 167) and `_air_keep = randf_range(0.75, 1.7)` (line 146). Attack (and therefore the swoop) only triggers at horizontal `dist <= attack_range` (6.0 for this cell). So whenever `_air_keep > ~1.18` — **a ~55% coin flip at spawn** — the dragon's own AI parks it permanently outside its attack trigger, and it never initiates; even good rolls stall after the keep-away pushes it back out post-swoop. A melee player (reach 2.4 m horizontal) cannot start the fight either — only walking directly under the hover point transiently triggers it.

**Fix direction:** clamp the aerial stand-off below the attack trigger, e.g. in `enemy.gd` where the flyer slot is computed: `keep = minf(keep, attack_range - 0.5)` (or cap `_air_keep` so `keep <= attack_range * 0.9` when the cell authored a boss `enemy_range`). Data-only mitigation (raising `enemy_range`) does NOT work — `surround_radius` scales with it, so the ratio and the coin flip stay the same.

## ❗ P1 — The dragon renders ~2–2.5× oversized and eclipses the portrait frame at close range

**Symptom (observed):** at first contact the dragon's torso/leg fills ~85–90% of the portrait screen for several seconds with the hero a sliver underneath (`p400_hud0`, `p400_premove`, `f400_t0`); one swoop frame (`f400_f2`) shows a wing+talon slab covering the upper half of the frame. It reads exactly like the "monster popup" failure mode. (The camera never clipped *inside* the mesh in any frame — the SpringArm ignores enemies and the near-fade hides it at true point-blank — so this is P1, not P0.)

**Root cause (verified by measuring the GLB):** `models/meshy/mire_dragon.glb` is a normalized rig — skinned mesh AABB **0.48 m tall × ~1.0 m long**, bone Y-span **0.36 m**, global scale 1. `enemy.gd` scales the model **to `enemy_height` by its HEIGHT** (`m3.scale *= max_h / mh`, ~line 218). Height is a horizontal dragon's *smallest* dimension: scaling to 4.5 m tall makes it **~9.4–12.5 m long/winged** (×9.4 if it measures the mesh envelope, ×12.5 if the bone span wins). That matches the frames — next to the 1.73 m hero it reads as a 10 m+ colossus, not the authored 4.5 m. Consequently the near-camera fade (`set_camera_near`, threshold `max(1.6, 0.45 * body_h)` ≈ 2.0 m) is tuned to its *height* and never fires while a 10 m wing sits 3–6 m from the lens filling the screen.

**Fix direction (either/both):** (a) data: author `enemy_height` ≈ 2.0–2.5 for this model so the *span* lands near the intended 4.5–6 m bulk; (b) engine: key the near-camera fade on the model's largest AABB extent (or projected screen coverage) instead of `body_h` alone, so any oversized mesh fades before it walls a phone screen.

## ❗ P1 — The camera cannot look at the boss: pitch clamp (+14°) < the dragon's hover elevation (~30–50°), so most damage arrives from off-frame

**Symptom (observed):** during the engaged stretch (`f400_f3`–`f8`) the player's HP bar visibly drains for ~30 s with **no dragon anywhere in frame** — it hovers 5 m overhead just outside the top of the view, and the `dragon_aggro` beat itself fired with an empty sky on screen (`c400_c11`). On a phone this reads as invisible chip damage; no overhead-threat indicator is discernible in any frame.

**Root cause:** `main.gd` `CAM_PITCH_MAX := 0.25` rad (≈14° up — deliberately tight to protect the sky from drag-look). The dragon at `enemy_hover` 5.0 m and 4–9 m horizontal sits at 30–50° elevation: **the orbit camera physically cannot frame it** until it is >~20 m away or mid-swoop. Every other enemy in the game is grounded, so this only bites the new encounter.

**Fix direction:** any one of — raise the pitch clamp toward ~0.6 rad when an aerial enemy is engaged; lower `enemy_hover` (e.g. 2.5–3 m) so the hover sits inside the reachable frame; or add a brief camera lift/target-bias while the boss is within its `show_within` radius. (Melee itself is fine — the hit test ignores Y (`to.y = 0`), so swoop-timed melee connects; the problem is purely that the player can't *see* the thing hitting them.)

---

## ✅ Touch controls — one-handed play works via the real touch path

Verified with CDP `Input.dispatchTouchEvent` (no mouse): left-half joystick drag moves the player (`p400_premove`→`postmove`, world + minimap moved); right-half drag orbits the camera (yaw change `postmove`→`postlook`); ATTACK tap fires the melee swing (mid-swing pose captured), SHEATHE tap toggles to DRAW with the weapon visibly stowed, JUMP tap goes airborne, POTION taps accepted. Button grid sits in the right-thumb zone, joystick zone owns the left half — no cross-eating observed.

## ✅ HUD fits phone portrait AND landscape — no clipping, no overflow, no debug text

`GOGI_HUD_GRID` portrait: vp 720×1548, rows 1064/1212/1360 — all on-screen; landscape: vp 1548×720, scale 1.78, rows 345/457/568 — all on-screen (`l860_postmove`). Short-side UI rescale works (buttons same physical size both orientations). Quest tracker, HP bar, minimap, BOUNTY/PLACES SEEN counters all inside edges. No fps/coordinate/debug text ships (`hud_debug` off, GOGI lines are console-only, director hides the stats block).

## ✅ Transient vs persistent UI — nothing pinned

"The Dragon's Roost" region name is a **toast (2.2 s hold + fade)** via `_toast(best, 2.2)` in `game_shell.gd:_update_region` — confirmed gone in later frames (`f400_calm0/1`), not pinned. Roost-entry subtitle (hold 5 s) and aggro subtitle both cleared on schedule (`f400_calm0/1` show a clean HUD). Apparent long toast lifetimes in early frames are the software-GL time dilation, not a decay bug — decay is delta-driven and completes.

## ✅ Feedback is non-modal; shake amounts are safe

`player_damaged` → 0.15 cam kick + HP bar, no popup/dialog. Rule shakes 0.3/0.45/0.35 map to ≤±0.16 m camera offset decaying in ~0.2–0.5 s — a firm thud, nowhere near nauseating. `roost_warn` (subtitle+shake 0.3) and `dragon_aggro` (thunder+subtitle+shake 0.45) both fired live and read well.

## ⚠️ Polish

1. **Landscape subtitle underlaps the right thumb grid** — subtitle box spans to x=0.78·vp while the button columns start at ~0.68·vp, so the roost-entry line renders under/over the SHEATHE button (`l860_postmove`). Narrow the subtitle to ~0.5·vp (or end it at the button column) in `game_shell.gd:_relayout`.
2. **Portrait region toast overlaps the quest tracker** for its ~3 s life (toast at y≈0.16·vp sits inside the quest block at y 140–340vp) — two yellow text blocks interleave illegibly at the exact moment you arrive somewhere new (`p400_hud0`). Nudge toast y below the tracker or hide the tracker while a toast shows.
3. **No HP feedback for a 700 HP boss** — `director.boss` only knows `murk_reaver`, so the Mire Dragon gets no boss bar; the only mid-fight state cue is the one-shot enrage subtitle. Consider generic boss-bar support keyed on authored `enemy_height`/`enemy_hp`, or a second boss entry.
4. **Stats debug-ish block flashes at boot** — "Lv 1 HP … Inv: [Reed Torch]" is visible for the first seconds until rules load and `hide_hud: ["stats"]` applies (`l860_hud0`). Apply `hide_hud` before first frame if cheap.

## Could not verify (sandbox limits)

- **Enrage (half-HP) and kill/hoard payoffs live** — the engagement P1 above meant no boot produced 350+ dmg dealt; even with a low-HP copy the dragon stayed out of melee reach. Beats are well-formed in `world.json` and use the exact subtitle/shake/toast paths proven above, but the enrage beat, kill toast, Dragonfang chest, and death-crash feel were not rendered. Re-verify after the engagement fix.
- **Mirewood rumor toast in situ** (spawned at the roost, not Mirewood) — rule present and `once`, toast path proven.
- Thunder SFX (container has no audio device), true multi-touch feel (joystick+button simultaneously — single-sequence touch only), and real-device notch/safe-area (engine has explicit safe-area handling with a 12% clamp; web tier unaffected).
