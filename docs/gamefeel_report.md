# Game-Feel / Mobile-UX report — "Heronwade" CONTINUE build (Hydra Dragon delta)

**VERDICT: PASS — 0 P0, 0 P1 in the session delta; 2 new ⚠️ polish items + pre-existing items unchanged.**

Scope: second boss `hydra_dragon` @ cell [-6,10], "The Dragon's Roost" + mobile HUD/touch regression.
Method: drove MY OWN offline copies of the canonical export (`/workspace/gamefeel-out`, mp-off + start_cell [-6,10]; low-HP variant `/workspace/gamefeel-out-lowhp` for the kill beats — repo/out untouched) in software-GL Chromium at **portrait 400×860 AND landscape 860×400**, all gameplay via **real CDP touch events** (no mouse/keyboard), 6 boots. Evidence: `/workspace/gamefeel/shots/*.png`, console logs `/workspace/gamefeel/logs/*_console.log`, probe script `/workspace/gamefeel/gf_probe.mjs`. Software-GL dilates game time ~2–3×; timings below are wall-clock.

---

## ✅ CAMERA vs the new boss — never walls the frame, never clips inside it (both dragons compared)

- **Melee-range framing:** the hydra chased and hit within ~3 s of region entry in every boot (`dragon_aggro on=player_damaged` at +0.3 s after `roost_warn`, 6/6 boots). At point-blank the boss covers ~35–50% of the portrait frame with hero, ground, sky and HUD all readable — an imposing boss, not a "monster popup" (`P3_t1_contact`, `L2_t2_melee`). Three heads + blue/red scales read clearly on a 400px-wide screen (`P3_t1_contact`).
- **Full 360° orbit at melee (8×45° touch drags, boss pileup at the lens):** no angle walls the view, and no frame renders the camera INSIDE a mesh — the SpringArm ignores enemy layer and the near-fade never needed to fire; worst case is the boss body briefly occluding the hero, which stays readable (`O_orb0..7`, esp. `O_orb1`, `O_orb5`).
- **Drag-look stays usable mid-fight:** right-half touch drag changed cam_yaw 0→3.07 rad (landscape) / 0→1.43 rad (portrait) while being hit (`L2_t3_afterlook`, `P3_t3_afterlook`).
- **Pitch clamps hold:** max down-drag still shows bosses + ground context, not feet (`P3_t4_pitchmin`); max up-drag keeps the horizon mid-frame, no sky-stare (`P3_t5_pitchmax`). The last session's pitch-vs-hover P1 is moot — the hydra is grounded (no aerial/hover keys) and engages instantly, so both prior engagement/oversize P1 classes are confirmed FIXED-not-regressed for the new boss.
- **Comparison vs mire_dragon one cell west:** same behavior — chases, melee framing fine, no wall/clip (`L2_t9_mire_melee`, `L2_t11_latehud`, and side-by-side in `P3_t5_pitchmax`, `O_orb1`).

## ✅ TOUCH controls one-handed, mid-fight (real touch path, portrait AND landscape)

- Joystick (left-half touch drag) moved the player 18.5–20.5 m WHILE the bosses shoved them around (`JOY check` in both logs).
- **ATTACK tap kills the boss end-to-end:** in the low-HP probe copy, repeated touch-taps on ATTACK drove real damage — `hydra_enrage on=enemy_hp_crossed` (0.5) then `bounty_hydra on=kill` fired, BOUNTY 0→350 on the HUD (`K_k_enrage`, `K_k_kill`, log `K_console.log`). Mid-swing pose captured (`P3_t6_attack`).
- JUMP tap → vy +5.93, anim `jump` (landscape; portrait re-verified vy>0). USE / POTION taps accepted. Button grid sits in the right-thumb zone in both orientations (portrait rows y 591–838 css of 860; landscape rows 194–372 css of 400) — nothing off-screen or under a notch inset.
- No cross-eating observed: HUD-button touches never turned into camera orbits and vice versa (one probe artifact confirmed the guard works — a drag STARTED on POTION was correctly claimed by the button, not the camera).

## ✅ HUD fits phone portrait AND landscape — no clipping, no overflow, NO debug text

`GOGI_HUD_GRID` portrait: vp 720×1548, rows 1064/1212/1360 (all on-screen); landscape: vp 1548×720, rows 345/457/568 (all on-screen); insets 0. Frames confirm: quest tracker, HP bar, minimap (bordered, with margin — not clipped, boss shows as red dot), BOUNTY/PLACES SEEN, 6-button grid all inside edges in both aspects. Title screen fills portrait cleanly (`P_t0_spawn`). No fps/coords/debug text in any of ~25 frames; the delta's new `GOGI_CAPTURE`/`_query_num` code is console-only and gated behind `?capture=1`.

## ✅ Transient vs persistent UI — every new beat fades, nothing pinned

Region title "The Dragon's Roost" = toast, gone a few seconds later (`L2_t1_contact` → `L2_t2_melee`). `roost_warn` subtitle, `dragon_aggro` subtitle, `hydra_enrage` subtitle ("All three heads scream…"), and the `bounty_hydra` kill toast all cleared on schedule; 25 s-later frames show a fully clean HUD in BOTH aspects (`L2_t11_latehud`, `P3_t11_latehud`, `K_k_postkill`).

## ✅ Damage feedback is non-modal

Hits = HP-bar drain + red flash + camera kick; no dialog/banner in any frame across ~200 boss hits taken. Enrage/kill beats are subtitle/toast + shake, also non-modal.

---

## ⚠️ Polish (new this session)

1. **Size hierarchy reads INVERTED: the tougher new boss looks smaller than the old one.** Side-by-side at melee, the mire_dragon clearly dwarfs the hydra (`P3_t5_pitchmax`, `O_orb1`) despite the hydra being the "bigger" boss (hp 900 vs 700, dmg 24 vs 20, bounty 350 vs 300, enemy_height 2.5 vs 2.2). Root cause (measured from the GLBs): `enemy.gd` scales to HEIGHT, and the hydra mesh is proportionally taller (0.683 tall × 1.0 long vs mire 0.48 × 1.0) — so hydra scale ×3.66 → ~3.7 m body / 3.2 m wingspan, while mire scale ×4.58 → ~4.6 m body / 4.5 m wingspan (~35% more bulk). Fix direction: raise the hydra cell's `enemy_height` to ~3.0 in world.json (bulk then edges out the mire; near-fade threshold unaffected — max(1.6, 0.45·3.0)=1.6 m, and the bulkier mire already proved this envelope safe at the lens).
2. **Both bosses always converge into one 2-v-1, and the second arrives without a cue.** `enemy.gd` chases from ANY distance once the cell is resident ("DETECTION IS REPORTED, NOT ENFORCED", line ~563), and the lairs are one cell (16 m) apart — in 3 of 4 fight boots the mire_dragon crossed into the hydra fight within ~60–90 s (`K_k_enrage`: mire at point-blank mid-hydra-kill; `P3_t4/t5`, `O_orb1`). Combined ~44 dmg/cycle vs 100 HP drove the bar to a sliver (`O_orb7`); the `dragon_aggro` stinger is `once`, so dragon #2 lands hits with zero on-screen announcement. Not a blocker — it matches the authored fiction ("TWO dragons roost") and the engine's forgiving heal-to-full death recovery caps the frustration — but consider either a second aggro beat keyed on the hydra/mire individually, or one more cell of separation, if the tag-team is not intended.

## Pre-existing (from repo docs/gamefeel_report.md + qa_report.md — verified unchanged, NOT re-flagged)

- Subtitle box underlapping the right thumb grid: still present, unchanged (landscape enrage line runs under SHEATHE, `K_k_enrage`; portrait aggro line wraps under JUMP, `P3_t1_contact`). Same class/severity as documented.
- Toast overlapping the quest tracker's first line (kill toast, `K_k_kill`). Same class as documented.
- Stats block ("Lv 1 HP … Inv:") flashing at boot until `hide_hud` applies (`L_t0_spawn`). Unchanged.
- No boss HP bar (the hydra, like the mire, gets no bar; only the one-shot enrage subtitle marks progress through 900 hp). Documented last session; the second boss makes it marginally more noticeable but it's the same defect.

## Could not verify (sandbox limits)

- True multi-touch feel (joystick + button held SIMULTANEOUSLY) — CDP drove single-sequence touch only; real-device notch/safe-area; audio stingers (no audio device); real-GPU frame rate.
- **Lair dressing in situ**: the bone/skull/ribcage/dead-tree props are CDN-only (`/godot-assets` 404 locally per the known artifact), so the roost rendered as bare dirt in every probe frame — prop framing/occlusion vs the fight could not be judged. Re-check one frame on the preview URL if dressing matters for the ship call.
- Supabase/multiplayer HUD paths (mp disabled per the known TLS constraint).
