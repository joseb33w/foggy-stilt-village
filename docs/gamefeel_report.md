# Game-Feel / Mobile-UX Report — Heronwade (feat/combat-vehicles-expansion)

**VERDICT: FAIL (0 P0, 2 P1 must-fix)** — touch play is fully alive and the HUD fits both
orientations, but the headline feature of this build (the boss encounter) ships with a walled
camera and an illegible HUD collision in portrait. Both are must-fix before this reads as done.

Method: drove the real web export headlessly through the sign-in gate using **CDP touch events
only** (no keyboard/mouse for gameplay verbs), at 720×1280, 390×844, 844×390 and 1280×720.
Evidence screenshots + full console logs: `/tmp/gf/*.png`, `/tmp/gf/log-*.txt`.
`out/world.json` was restored to the pristine build file afterwards. Note: another agent was
concurrently rewriting `out/world.json` (a port-5293 probe variant); my harness serves its probe
world from memory, so my results are race-free, but coordinate future probe scheduling.

---

## ❗ P1 — Boss-fight camera is walled by the Murk Reaver at melee range (portrait)

**Symptom** (`/tmp/gf/bossmelee-m1.png`, `bossmelee-m2.png`, `boss-b1-portrait.png`): once the
5 m boss (`enemy_height: 5.0` at cell [13,12]) closes to melee, its mesh fills **~85% of the
portrait frame** and the player character is completely invisible — reproduced across repeated
ATTACK taps 4 s apart, i.e. this is the steady state of every melee exchange with the boss, not a
transient. The screen reads as a dark full-frame smear / "monster popup". Normal 1.8 m enemies are
fine (`combat-c3-end.png`: two bandits at point-blank, clean framing) — this is specific to the
oversized boss.

**Root cause**: two mechanisms exist but neither covers a 5 m body:
- `main.gd` `cam_spring.collision_mask = L_WORLD` — the SpringArm never collides with enemies, so
  the camera can sit against/inside the boss volume.
- `enemy.gd:695` near-camera fade hides the mesh only when camera-to-body-centre
  `<= max(CAM_FADE_NEAR=1.6, 0.45*body_h)` = **2.25 m** for the boss — but a 5 m-tall, several-m-wide
  body walls a portrait frame from 4–8 m away (camera rides at `CAM_DIST=8.5` behind the player,
  who stands at 2.4 m melee reach). The threshold never trips during a normal exchange.

**Fix direction**: for bodies over `MAX_ENEMY_H` (the authored-giant path), fade/dither by distance
to the mesh **AABB surface** instead of the centre (or hide when the camera is inside the AABB
inflated ~1 m), and/or raise the fade multiplier for giants (0.45 → ~0.9 of body_h), and/or
auto-raise camera pitch when the acquired melee target is a giant. Do NOT fix by shrinking the boss
— the 5 m silhouette at approach range (`boss-b2-landscape.png`) is good drama; only melee framing
is broken.

## ❗ P1 — Boss bar collides with the stats line and the BOUNTY readout (portrait)

**Symptom** (`boss-b1-portrait.png`, `bossmelee-m1.png`, and the coordinator's own
`probe-game.png`): within 60 m of the boss at 720×1280, "**THE MURK REAVER**" overprints the
top-left stats line — "XP 0/30 Gold 0" becomes illegible garbage — and the **BOUNTY 0** readout is
printed directly ON the boss HP bar, obscuring both. Three HUD elements stack in one band.
Landscape is legible (`boss-b2-landscape.png`) — this is portrait-only.

**Root cause**: `game_shell.gd::_relayout()` (line ~850) pins the boss name at y=12 / bar at y=44,
top-centre — the same band `main.gd::_relayout_ui()` gives the stats block (`stats.position =
(ml, mt)`, 3 lines of 22 px text ending ~x 360 in portrait, so the centred name overlaps it). And
`rules.gd::_engine_hud_rects()` collects buttons/stats/hp/minimap but **omits every GameShell
surface** (`_boss_root`, `_quest_lbl`, `_toast_lbl`), so `_place_clear()` believes (372, 38) — the
middle of the boss bar — is free and puts the BOUNTY readout there. The "GOGI_HUD_FIT UNRESOLVED"
tripwire never fires because the fitter can't see the obstacle it's overlapping.

**Fix direction**: (a) add the shell's visible surfaces (at minimum `_boss_root`) to
`_engine_hud_rects()` so world readouts dodge the boss bar; (b) in portrait, drop the boss bar
below the top text band (e.g. `y = stats bottom + 8` instead of the hardcoded 44/-32), or
right-size `_boss_name` width to the bar and give it a shadow/background so it can't overprint.

---

## ✅ Touch controls (the P0 class — verified alive, touch-only)

- Sign-in gate: fields + Sign-in button all operable via touch taps.
- **Joystick** (left-half drag-hold): 20.8 m on foot; drives all three vehicles (buggy 20.9 m/5 s,
  skiff 41.9 m/9 s, Dragonfly 83.8 m/9 s). One earlier 0.4 m reading was the buggy parked
  nose-against-an-obstacle (keyboard moved it 0.06 m in the same state) — environmental, not input.
- **Drag-look** (right-half): 1.56 rad on foot, works mounted on all three vehicles; pitch clamps
  hold at both extremes — no scalp/sky stare (`base-p3`, `base-p4`).
- **Buttons** via touch: ATTACK lands kills (kill rules fired touch-only), JUMP leaves the ground
  (vy=5.9), SHEATHE⇄DRAW relabels and stows the torch (`base-p5`), DISMOUNT appears on board,
  sits in the thumb grid (row-3 left column), and exits car/boat/plane. Button taps do NOT hijack
  the camera (`_touch_on_hud` guard works).
- **Two-thumb multi-touch**: simultaneous move (30.3 m) + look (1.44 rad), and the left thumb keeps
  driving after the right lifts. (CDP-emulated; see caveats.)

## ✅ HUD fits phone aspects; no debug text

720×1280, 390×844 (small portrait), 844×390 (short landscape), 1280×720 all render every control
on-screen, no clipping, no button under the half-line (`base-p6`…`p8`). The short-side UI rescale
(`_fit_ui_scale`) keeps buttons/text at designed size in landscape. No fps counters, coordinate
dumps or netsync overlays ship — `?hudgrid=1` / `?mpdebug=1` / `?soak=1` are opt-in and off. The
`Inv:` line truncates at 43 chars + "…" so a full 6-weapon inventory can't overflow the width.

## ✅ Transient vs persistent UI

Region names ("The Reed Road", "The Broad Murk") are 2.2 s toasts that fade — verified gone 8 s
later (`base-p1` vs `base-p2`). No pinned zone label. Interaction affordance ("USE > Drive
Dragonfly") is contextual. Victory/damage do not pin text.

## ✅ Damage feedback is non-modal

HP 100→34 under a bandit pack with zero popups/banners: feedback is directional hit-marks + camera
kick (the full-screen red flash was deliberately removed, `main.gd::_flash_hurt`). No modal fired
in any run (no `GOGI_MODAL` outside authored dialogue).

## ⚠️ Polish (non-blocking)

1. **Quest tracker verbosity**: the full 4-objective checklist (9 wrapped lines of yellow text)
   is pinned top-left for the whole session (`base-p1`), covering ~¼ of the portrait play view in
   fog-heavy scenes. Consider current-objective-only (the data is per-step in quests.json).
2. **Readout placement drift**: BOUNTY is authored `top_left` but the collision-dodger relocates it
   per-context (usually top-centre at (372,38); bottom-left when crowded); PLACES SEEN (`top_right`)
   lands bottom-right, 8 px under the ATTACK button where a resting thumb covers it. Functional,
   never overlapping (except the boss-bar case above), but the two build-advertised readouts are
   not where a returning player last saw them.
3. **Fly dismount mid-air**: tapping DISMOUNT during the take-off run exited with the player
   briefly mid-air beside the plane (`fly-v3-after-dismount.png`) before dropping into the bay.
   `exit()` is documented as braked-descent-safe and the player landed in water; worth one native
   sanity pass.

## Could not verify (sandbox limits)

- Real multi-touch digitizer feel (CDP-emulated touch only), haptics, and true frame pacing on a
  phone GPU (this container is software-GL).
- Notch/safe-area insets: `_safe_insets()` clamps correctly in code, but the web container reports
  zero insets, so the notch path was not exercised with real values.
- Supabase reachability from a real device (container TLS artifact forced the local /sb proxy).
