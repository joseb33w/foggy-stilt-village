# QA Report — "Heronwade" combat/vehicles/boss expansion
**Build:** joseb33w/foggy-stilt-village, working tree at `/workspace/repo` (see P1-2: the named branch does not exist), export `/workspace/repo/out`
**QA method:** independent adversarial pass — fresh canonical verifier download + run, static world/quests/engine analysis, headless-Godot GLB clip checks, and 12 of my own auth-aware browser probes against the export (screenshots in `/workspace/verify-qa/`, logs `/tmp/qa-*.log`, verifier log `/tmp/verify-qa.log`). `out/world.json` restored byte-identical after probing.

---

## VERDICT: FAIL (1 P0)

The build is rich, well-wired and almost entirely healthy — but **the final boss fight is dead**, and the boss kill is the literal win condition (`goal: complete_quest rout_of_the_reeds` → `kill murk_reaver ×1`). As shipped, the game cannot be won.

---

## ❌ P0-1 — Murk Reaver never appears, never engages, cannot be hit: the game is unwinnable in practice

**Evidence (4 independent sessions, ~6 min total at/near the boss):**
- The boss bar "THE MURK REAVER" shows full — and per `game_shell.gd:_update_boss_bar()` the bar is only visible when a **live** `murk_reaver` instance is within `show_within` 60 m → the boss **does spawn and is alive**.
- Enemy placement is deterministic (`chunk_manager.gd` ~line 1196): a single enemy spawns on a ring r = `half*0.45` ≈ **3.6 m from the cell centre**, i.e. ≈ **(219.6, 200)** for cell [13,12]. My probe walked the player to that exact point and **circled it at 2.8–4.9 m for 8 legs, then clicked ATTACK for 60 s** (Heron Talon Blade equipped, 60 dmg): **no reaver visible anywhere** (a 5 m-tall boss!), **no `player_damaged`**, **no `bounty_reaver`**, HP 100/100 both ways (`qa-boss-at-spawnpoint.png`, `qa-boss-end.png`, `/tmp/qa-boss6.log`).
- Standing on the dry island top 25–35 m away for 110 s: boss never approached (its minimap dot never moved), despite `enemy.gd`'s explicit "creatures chase from any distance" design (`/tmp/qa-boss5.log`, `qa-boss-t1.png`).
- **Root cause:** cell [13,12]'s centre area is **flooded** — my `gy` readings around the spawn ring were **−1.4 … −4.1 m vs water level −0.3** (the cell sits on the canyon-carved NE slope of the delta cone). The reaver spawns into 1–4 m-deep water/ground and ends up invisible (below the opaque water/terrain), immobile and out of everyone's reach. Contrast: rats/pirates/lurkers, whose cell centres are dry, all spawn, chase and fight correctly (verified — see passes below).
- Collateral in the same cell: the **Heron Talon Blade chest at [13,12] pos [4,4] ≈ (220,204)** is placed at `_ground_y` → also 1–4 m underwater; likely un-openable (minor, the blade is also gifted by the `talon_gift` rule).

**Note:** `qgcheck` reports the world winnable — the quest **graph** is fine; this is a physical/placement failure qgcheck cannot see.

**Fix direction:** put the boss's spawn ring on dry land — e.g. move the boss (+chest) to a cell whose centre is on the island top (cell [12,11], centre (200,184) = cone summit, height ≈ +15), or add a terrain feature (basin-inverse / raise) so [13,12]'s centre clears water, or author `enemy_aquatic: true` + a larger `enemy_range` so a flooded reaver still swims up and engages. Then re-verify end-to-end: point-blank engage (`player_damaged`), kill (`bounty_reaver` fires, bar drains/empties), talon chest reachable, and the `rout_of_the_reeds` → victory panel chain.

---

## ❗ P1 issues

**P1-1 — Boss-fight verification debt behind P0-1.** Because the boss never engages, the following ship-claims remain **unverified**: reaver model/animation in-game (GLB itself is healthy: `idle/walk/attack/death` + skeleton + real textures, verified headless), boss-bar drain on hit, `bounty_reaver` toast, quest-4 completion, victory panel + victory music. All are engine-standard machinery (kill→rule chain proven with rat_bandit), but after the P0 fix the whole boss loop needs one real kill-run.

**P1-2 — Delivery/branch state: `feat/combat-vehicles-expansion` does not exist; the expansion is uncommitted.** The repo is on `main` at `08297e1` with the entire expansion sitting as **uncommitted working-tree changes** (world.json +21 853 lines modified, quests.json, README, meshy_assets.jsonl, ~14 untracked new files incl. `models/meshy/*.glb` for all combat/vehicle assets and 3 new audio tracks). `git branch -a` shows only `main`. One crash/reset loses the build; the named PR branch can't be reviewed. Commit + push before ship.

---

## ⚠️ Polish / notes (non-blocking)

1. **Skiff nose metric ambiguous:** boat boards, travels 8.5 m and exits cleanly, but `veh_nose_dot_fwd ≈ 0` (drive = 1.0, fly = 0.99). Visually the skiff reads bow-forward with the player standing amidships (`qa-boat-seated.png`) — likely the fused GLB's authored axis vs the mount yaw. Worth one on-device glance that it doesn't travel broadside.
2. **Plane rider fully hidden:** boarding the Dragonfly the wanderer disappears into the fuselage (`qa-fly-seated.png`) — acceptable "closed cab", but a visible head/canopy would read better. Buggy is correct (rider visibly seated in the cab, `qa-drive-seated.png`).
3. **Vostok props render near-white** (MS_Tent, MS_Crate, MS_Plank_Pile at camps/anchorage) — reads slightly untextured under fog + software-GL (`qa-anchorage-a.png`); expected to look right on device, but check once.
4. **Bog lurkers are near-silhouette dark** in fog (`qa-mirewood-a.png`) — dramatic but borderline readability; a touch of rim/emissive or lighter albedo would help.
5. **Reed Runner spawns in cell [9,6] with 2 aquatic pike_pirates** — boarding it is contested; if intentional (ambush), fine.
6. **Title wraps "HERON WADE"** on two lines in portrait (`probe-title.png`) — cosmetic.
7. Coordinator's `luma-night.png` is actually the sign-in gate, not a night frame — harmless (sky is pinned `{time: day, weather: fog}`, no night phase exists), but the artifact is mislabeled.
8. Worst frame 233 ms (under the 250 ms bar) — acceptable; average-fps numbers are container artifacts, ignored.

---

## ✅ What passed (with the evidence read)

| Dimension | Result | Evidence |
|---|---|---|
| Boot / console | ✅ | verifier: engine booted, canvas, **console clean**; scene-instantiation 35/0 failed |
| qgcheck winnability (graph) | ✅ | "quest-graph OK — world is winnable (576 areas)" (but see P0-1) |
| Kill-count feasibility | ✅ | quests need 6/3/4/1 kills vs authored 10 rats / 6 lurkers / 8 pirates / 1 reaver |
| Rules layer | ✅ | `start_amb` fired post-auth in **every** probe; `seen_*` region rules fire; coordinator's kill/bounty/hurt rules re-confirmed (rat chain) |
| Movement + camera | ✅ | 5.5 m on W; drag-look 1.08 rad; camera collision works (no wall clipping) |
| No autofire-on-look | ✅ | attack is a dedicated button; look-drag fired zero attack/kill rules (asserted); HP/bounty unchanged |
| Combat: enemies engage + damage both ways | ✅ | rats: HP 46→10 between frames, surrounded player (`qa-camp-*.png`); lurkers: HP 16→44 (died + **in-place respawn, no modal death UI**); pirates converge (`qa-anchorage-*.png`); kill chain (attack→death→`bounty_rat`+`first_rat`) from coordinator's combat probe |
| Enemy models / clips / no T-pose | ✅ | all 4 combat GLBs: idle/walk/attack/death + skeleton (headless check); rats/pirates/lurkers textured + animated + correct facing in-game; **0 GOGI_PLACEHOLDER** once CDN proxied (in-container placeholder lines are a chromium-TLS sandbox artifact — asset URLs all 200) |
| Enemy stats sanity | ✅ | 70/6, 180/14, 110/9, 480/18 authored per cell; 55 % per-hit cap → min 2 hits on everything; talon 60 dmg → 8-hit boss |
| Weapons catalog + chests | ✅ | 6 weapon defs match chest contents at [4,2],[3,-4],[-6,3],[12,2],[13,12] + `talon_gift` rule on `clear_the_waters`; start weapon = restatted `rusty_sword` → "Reed Torch" 18 dmg (renders in hand) |
| Weapon-in-hand | ✅ | torch gripped (multiple shots); talon blade gripped at boss (`qa-boss-island.png`); bow/staff are parametric (not visually sampled — low risk) |
| Vehicles board/travel/exit | ✅ | all 3 profiles via the real `try_use` path; buggy nose-first dot=1.0, upright; plane dot=0.99, airborne dy=2.6 (coordinator) ; boat rides water y=−0.23; DISMOUNT affordance appears |
| World persistence / boundary | ✅ | walked into the west border cell: **invisible edge wall stops the player at x≈−127** (78 s of held W, x unchanged); world keeps rendering, engine alive (`qa-edge-offgrid.png`) — no world-vanish possible |
| World richness / density | ✅ | land side: 360 cells, **avg ~41 entities/cell, 0 empty**; village byte-preserved (64 cells, only the disclosed [4,2] chest added — `check_preservation.py`); roads render on the causeway; camps have tents/campfires/crates; anchorage has props+structure+chest; boss delta has skull-totems/graves. Verifier SPARSE warn (~1.9/cell) is bay-dominated: 216 open-water cells at 0.9 — intentional boat space |
| Art / style | ✅ | foggy muted swamp reads as authored: textured mud/sand/grass grounds up close, fog-graded horizon, no grey ceiling slab (fog-white sky is the weather), textured hero (bearded wanderer in ochre oilskin) + heron NPC models, feet on ground (`foot_raw` ≈ 0.05) |
| Mobile fill / HUD | ✅ | portrait 720×1280 and 400×860 + landscape 860×400 all full-bleed, four corners, HUD inside, no overlaps; `GOGI_HUD_FIT` auto-moved bounty/explored to the bottom corners to avoid the stats stack |
| Budgets | ✅ | pck 11 MB; GPU peak observed 82 MB video / 33 MB tex (of 220 budget) incl. camp/anchorage/boss areas; LIVE_ENEMY_BUDGET caps skinned enemies |
| Native tier | ✅ | `manifest.json`: `webOnly: false`, world.json data-driven, requires rules+hud |
| Audio presence | ✅ | AudioManager + bus layout + 20 tracks incl. new road/boat/tension; regions reference existing files; verifier audio pass silent (no warn) |
| Meshy sourcing | ✅ | every character/creature/vehicle is Meshy (`models/meshy_assets.jsonl` prompts match shipped GLBs); kk_* items are props/rig-donors only; hero/NPC `/cloud-iybqouv5yf6dymalatmc/…` URLs are self-healed by `main.gd:_norm` onto the current build id (files present locally — works) |
| Preservation | ✅ | original 64 village cells byte-identical except the disclosed chest |
| Tofu / debug text | ✅ | none in any frame; apostrophes render |

## Could not verify (sandbox limits)
- Real-device GPU fidelity (vostok prop whiteness, lurker darkness — flagged above as polish), audio playback, touch feel.
- Browser↔Supabase realtime (in-container chromium cannot TLS to supabase/CDN at all — even plain fetches fail; the transport was proven from node by the coordinator's 2-client test, and auth works through the local proxy in all 12 of my probes). True 2-client sync, host-election and cross-area cull are untestable here.
- Full quest-chain playthrough (talk/collect/reach steps of quests 1–3) — machinery is engine-standard and rules fire, but no end-to-end run was driven; the boss re-verify after the P0 fix should walk the chain's final step anyway.
