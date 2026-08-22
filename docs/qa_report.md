# QA REPORT — HERONWADE (peaceful exploration, godot-tmpl-rpg chunk world)

**VERDICT: PASS (0 P0 · 3 P1 must-fix · 7 warns)**

The build is content-complete, boots clean, is winnable end-to-end through the real input
paths, and the fantasy reads: foggy stilt village, heron people, ochre-coat wanderer, giant
sleeping catfish. No ship-blocker class found (no T-pose, no moonwalk, no gray-box world, no
dead quest, no console errors, no letterbox, world persists at its edge). Three P1s below are
authoring/wiring bugs the player will notice — fix before ship.

Evidence: screenshots in `/workspace/verify/qa/` (referenced below), harness log
`/tmp/verify.log` (VERIFY PASSED), headless probe transcripts (real game process,
`interaction.try_use()` / real quest system — no test-only shortcuts).

---

## ❗ P1-1 — All `regions[]` circles are offset (−8,−8) from the world content

**Symptom (verified live):** standing at the actual village square centre (40, 41.8) shows
**PLACES SEEN 0** (shot `41_square_t30.png`); the toast only fires ~3 m further west (shot
`43_talk_elder.png`). Worse, **"The Elder's Hut" region can never fire at or inside the elder's
hut** — it fired at (28.9, 22.4), open mud next to a *different* hut (shot `07_hut_door.png`
shows the "The Elder's Hut" toast over the wrong building). "Old Murk's Pool" cannot fire on
the natural swim line from the pier end (z≈40): min distance to its centre (94,32) along that
line is exactly 8.0 = its radius (strict `d < radius` in `game_shell._update_region`); it only
fired when I approached from the south at (93.3, 36.6) (shot `82_pool.png`). "The Reed Road"
does not cover the spawn point (11.3 m > r10) — a player walking straight east never gets it.

**Root cause:** region `center`s (authored in world.json) assume cell centre = `cell*16`
(e.g. Village Square [32,32] for cell (2,2)), but the engine centres cells at `cell*16 + 8`
(`chunk_manager.gd:656/958` — confirmed: player spawns at (8,40) for start_cell [0,2]; Elder
Sedge registers at (36.5, 42.6), chest at (33.5,33.5), door at (37.2,38.8)). Every region is
therefore ~11.3 m SW of its content. All five region rules DO fire correctly when their
(misplaced) circles are entered — proven live: seen_gate (verify run), seen_square + seen_hut,
seen_pier (`80_pier.png`), seen_pool + murk_stir subtitle + shake (`82_pool.png`), so this is
purely a placement fix.

**Fix:** add +8,+8 to all five region centers (and consider nudging "Old Murk's Pool" onto the
catfish at (102,40)). Re-check the terrain `features` too — same coordinate assumption; today
the canyon (x 91..101) happens to work, but Old Murk (102,40) landed ~1 m *outside* it on the
east bank (0.94 m shallows — visually fine, see P1-3/notes).

## ❗ P1-2 — Quest objectives are never shown on the HUD

**Symptom:** the auto-started quest "Village of Stilts" has **no visible objective text at any
point** — confirmed in every screenshot of every session (no quest line anywhere). The player
gets no "Find Elder Sedge…" guidance; only region toasts.

**Root cause:** `game_shell.gd` — `_quest_lbl.visible = true` only inside the mode-`chain`
branch of `_apply_mode()` (line ~532), and `_refresh_quest_lbl()` early-returns when
`_chain.is_empty()`. This world's `wander` mode has no `chain`, so the label stays hidden
forever even though `quest.current_objective()` returns the correct live checklist (verified
headless: text updates step-by-step through all four steps).

**Fix:** show/refresh `_quest_lbl` whenever any quest is active (drop the `_chain` gate), or
author the quest as a 1-stage mode chain.

## ❗ P1-3 — Skerrin (the pier fisher) stands half-submerged in the channel, not on the pier

**Symptom (shot `81_pier_end_look.png`):** the flagship pier NPC wades waist-deep in the water
beside the pier ("USE > Talk to Skerrin" over open water). His registered position is
(92.8, −2.34, 40.5) — the channel bed, ~2 m below water level (−0.3). The request's "heron
people stand around on the docks" degrades for exactly the NPC placed on the Great Pier.
(Elder + villagers on the square ground are fine.)

**Root cause:** NPC placement grounds to terrain height (`chunk_manager` `npos.y = _ground_y`),
ignoring the elevated pier deck above; his authored spot is over the carved channel.

**Fix:** ground NPCs via a physics ray (hit the deck collider), or move Skerrin's `pos` west
onto the plank run (x ≤ 88) where deck ≈ terrain.

---

## ⚠️ Warns / polish (P2)

1. **Streaming pop-in / late dressing.** At container fps (4–8, software GL) the village
   dresses 30–60 s behind a walking player: shot `40_square_t0.png` = player standing at the
   square in a white void; `41_…t30` ground+huts; `42_…t60` fully dressed with NPCs. Builds are
   per-frame so this scales with fps (a phone sees seconds, not a minute), but quest-critical
   interactables (elder/chest/door) only register when their cell dresses. Recommend building
   the player's own cell's interactables first. (Related real hitch: **FEEL perf worst frame
   283 ms** while streaming — device-independent, will stall a phone frame; slice the heavy
   cell-dressing phase further.)
2. **Combat stats + red HP bar leak into the first minutes** of a combat-free game (shots
   `20_square.png`, `40_square_t0.png`; faint on the title screen `01_title.png`). The
   `hud_hide` start rule only fires at `world_ready` → after the spawn ring builds (fps-scaled
   delay). Headless confirms they hide correctly once the rule runs. Fix: apply
   `director.hide_hud` at HUD build time, not only at the start event.
3. **Wandering villagers body-block the elder hut doorway** (shot `61_door.png`: a populate
   NPC standing in the open doorway) and walkway furniture (post rows / drying racks / torches)
   creates pinch pockets — my driver got physically wedged twice (e.g. (55.5, 41.2) between
   plank, post row and hut roof eave). A human can steer around/wait, but consider keeping
   `populate` wanderers out of the door zone and dropping colliders on minor posts.
4. **Spawn-area boardwalk reads derelict:** ground-conformed planks lie at jumbled angles with
   gaps (shot `02_spawn.png`) rather than a continuous raised walkway; mid-village sections on
   stilts look correct (shots `48_pier.png`, `55_portrait.png`). Fine if "rickety" is intended.
5. **Old Murk placement nuance:** he sleeps ~8 m east of the pier end on the far bank in 0.9 m
   shallows — reads beautifully (shots `83_at_murk.png`, `85_edge.png`) but is "beside", not
   "under", the pier, and the murk_stir subtitle can be missed entirely on the direct swim
   (see P1-1).
6. **kk_furniture GLBs re-fetched per placement** (bed/table/etc. requested 7× each; browser
   cache absorbs it — resource audit in `/tmp/qa_drive3.log`).
7. **Local-serve model 404s:** absolute `/cloud-…/models/*.glb` paths 404 on a bare local
   server before the engine's self-heal re-roots them (it did; models loaded). The live host
   serves all model URLs 200 (curl-verified) — env noise, not a build defect.

---

## ✅ What passed (with the evidence that would have turned red)

| # | Check | Evidence |
|---|-------|----------|
| 1 | Boot → tap → title "HERONWADE" (kicker/tagline/hints) → WANDER starts | `01_title.png`; game starts, `__gogiPlayer` live |
| 2 | Console clean (no GDScript/JS errors) | verify PASS + 4 QA sessions (only the env-noise 404s above) |
| 3 | Player: ochre-raincoat wanderer, torch in hand, ~1.65 m, animates (idle/walk/run/jump/swim clips present) | `02_spawn.png`, `41…`, swim=true at pier; GLB audit |
| 4 | Facing: S→face, W→back (no moonwalk) | `03_face_S.png` / `04_back_W.png` |
| 5 | Camera orbit: right-half drag | cam_yaw 0→1.44 rad (numeric) + pixel change (verify) |
| 6 | Movement: WASD works; keyboard+joystick paths share one input vector (`main.gd _keyboard_vec + move_vec`) | all traversal legs; verify move probe |
| 7 | Village richness: 12 authored huts (counted in world.json), timber walls + thatch roofs + warm windows, boardwalk + stilt posts, mud ground with texture, dense scatter, fog sky (grey mist, not void/red) | `42_square_t60.png`, `43…`, `48_pier.png`, `63_chest.png` |
| 8 | Heron NPCs: tall/thin/long-beaked/hunched, textured Meshy models (villager/elder/fisher), idle+walk anims, sane scale (~2 m vs 1.65 m player), no T-pose, no flat-tint blobs | `43_talk_elder.png`, `61_door.png`, `63_chest.png`, `81…` |
| 9 | Talk path: `USE > Talk to Elder Sedge` prompt in range; talk advances quest (talked.elder_sedge, talks counter +1) | `43_talk_elder.png` + headless probe (real `try_use`) |
| 10 | NPC speech is voice-only by design (TTS/LLM via npc.myapping.com; endpoint reachable — /chat responds); quest advance does NOT depend on it | code path + headless |
| 11 | Enterable hut: hinged door on the walkway face (door_face "n" = +Z = boardwalk side), opens/closes via USE, lit interior, furniture GLBs fetched, chest inside | `61_door.png` ("Close Door"), `62_interior.png` (player inside lit doorway), headless: door at (37.2,38.8) |
| 12 | Chest → heron_charm + 10 gold, `charm_note` toast rule fires, quest step ✓ | headless: charm=true gold=10, fired=[…charm_note] |
| 13 | Reach c5_2 quest step on entering the pier cell | headless + live (`80_pier.png` toast) |
| 14 | Old Murk: giant 7 m mossy catfish (GLB 4×1.65×7 m), visible above the waterline beside the pier, not floating, `USE > Talk to Old Murk`, talk completes step | `83_at_murk.png`, `84_talk_murk.png`, `85_edge.png`; headless talked.old_catfish |
| 15 | Full completion: quest → done, `fog_settled` flag, `all_done` panel **"THE FOG FEELS LIKE HOME"** visible | headless tree assertion (panel Label visible) |
| 16 | Regions/toasts/counter: all five `enter_region` rules + PLACES SEEN increments + murk_stir subtitle + shake fire (at the offset positions — see P1-1) | `43…`, `80…`, `82_pool.png`, `07…` |
| 17 | World persistence/boundary: invisible border wall at x=112, ray blocked at 111.5, live walk stopped at x=111.0 with the world (catfish/pier/huts) still rendered behind | headless probe + `85_edge.png` |
| 18 | Mobile fill: portrait 400×860 and landscape 860×400 — canvas == viewport exactly, HUD (JUMP/USE, PLACES SEEN, minimap) inside, no overlap, no letterbox | `55_portrait.png`, `56_landscape.png`, DIMS logs |
| 19 | HUD: combat controls hidden (attack/weapon/potion/stable/cycle absent; JUMP+USE only), no debug text (after start rule — see warn 2) | all in-game shots |
| 20 | Day/night lighting: night readable (blue moonlight, player/trees distinct), day not clipped (harness luma: night mean 49, day max 255 clipped 0.0%) | `luma-night.png`, `luma-day.png` (deterministic gogiSetTime probe) |
| 21 | Winnability gate: qgcheck green (64 areas, world winnable); scene-instantiation 35/35; packaging OK | `/tmp/verify.log` |
| 22 | Native tier: chunk `world.json` + manifest `webOnly:false` → iOS-playable | `out/manifest.json` |
| 23 | Audio presence: AudioManager + bus layout + mystical.ogg/crickets.ogg/wind.ogg shipped; rules wire ambient crickets at start, director music "mystical", fog weather wind bed | files in `out/audio/`, verify audio-infra OK |
| 24 | Meshy mandate: player + all 4 NPC characters + catfish are Meshy assets (meshy_assets.jsonl, all match:PASS); library GLBs only for props/furniture | model audit |
| 25 | Input-binding sanity: no fire/attack bound anywhere (no combat); look = right-half drag only; USE/JUMP are discrete buttons | project.godot (no [input] fire), HUD |

**Sandbox limits (could not verify here):** real audio playback (muted container), live TTS/LLM
reply content on-device, true-GPU visual fidelity (software GL renders paler/rougher — fog
density and mud texture judged only approximately), touch feel, real-iOS native runtime.
Note: several long Chromium sessions had the tab die mid-run (likely SwiftShader/container
memory, not reproducible via any game action; verify's own 4-min session and one 14-min session
survived) — flagging only as an observation, not a defect.

**Verify harness:** `node verify.mjs /workspace/out /workspace` → **VERIFY PASSED** (WARNs:
no roads[] — correct for a swamp village with boardwalk `rows`; flat-tint static lint — visually
disproven, characters render textured; 283 ms worst frame — filed as warn 1; 7/9 rules not fired
in the harness's short session — all fired across my sessions).
