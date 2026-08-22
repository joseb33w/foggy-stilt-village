# QA Report — foggy-stilt-village, multiplayer CONTINUE change
**Scope:** data-only delta (world.json `multiplayer` block, `mp_respawn` rule, model-path rebase, README) on the already-shipped HERONWADE build. Export: /workspace/repo/out, served locally (two runs: plain + production-shaped `/cloud-iybqouv5yf6dymalatmc/` prefix with `/godot-assets/` CDN-proxied). Signed in through the REAL gate UI via Supabase-HTTP-over-Node (existing test account; no new accounts created).

## VERDICT: FAIL (0 P0, 1 P1 must-fix)

The delta is largely sound — sign-in, netcode config, model rebase, winnability, and normal play all verified green — but the change's own headline feature (`mp_respawn`: "real death + respawn instead of silent auto-heal") empirically delivers a **blocking full-screen DEFEATED modal on top of the already-respawned player**, contradicting the rule's intent and the engine's stated non-modal design. Fix (or consciously accept/revert the rule) before deploy.

---

## ❗ P1 — player death shows a permanent blocking "DEFEATED / TRY AGAIN" modal over an already-respawned, healed player
**Evidence (empirical, real damage path):** I hot-injected a QA rule via the game's own 4s world.json poll (served modified data from my test server — repo untouched): `{damage: 9999}` on `time_of_day_changed`, tripped with the engine's `gogiSetTime` hook. Console: `GOGI_RULE_FIRED qa_kill` → `GOGI_RULE_FIRED mp_respawn on=player_died`. Screenshots `verify/qa11_death.png` / `qa12_after_death.png`: the mp_respawn toast "The marsh takes you... and gives you back." renders at the top **while a full-screen DEFEATED modal covers the game**, and it never clears — held-W movement after 4s+ = **0.00 m** (input blocked). Clicking TRY AGAIN recovers (7.7 m movement after, `qb2_after_try_again.png`) but runs `main.restart_run()` — a **full run reset** (vars, rule state, opening sequence), not a respawn.

**Root cause (code order, pre-existing engine files):** `main.gd::take_damage()` (~line 1280, same pattern in `set_player_health()` ~1414): `director.fire("player_died")` runs the rule **synchronously first** — `mp_respawn`'s `respawn` calls `respawn_player(true)` → `clear_defeat()`, which no-ops because `game_shell._lost` is still false — and **then** `_show_defeat()` puts the modal up with nothing left to clear it. `game_shell.clear_defeat()`'s own comment says a world answering `player_died` with `respawn` must not keep the modal ("Leaving the modal up over a living, moving player is wrong in any world that respawns") — the call ordering defeats that design.

**Reachability:** this world has no single-player damage sources, so it only fires on multiplayer PvP hits (`netsync.hit_taken` → `take_damage`) — i.e. exactly the mode this change ships. README explicitly advertises "hits all sync."

**Fix direction (coordinator's choice):**
- Engine fix (1 line-ish, main.gd): call `_show_defeat()` **before** `director.fire("player_died")` so a rule's `respawn` clears it; or suppress `_show_defeat()` when the world's `player_died` rules contain a `respawn` action.
- Data-only fallback: drop the `mp_respawn` rule — `wants_death()` reverts to false and the engine's non-modal heal-in-place + strong hurt-flash returns (pre-change shipped behavior). Loses the "real death" feature but ships clean.
- Do NOT ship as-is: a PvP death = blocked screen + run-resetting button.

## ⚠️ Warns / observations (not ship-blockers)
- **Uncommitted delta:** the repo has NO `feat/multiplayer` branch — the change exists only as uncommitted working-tree edits on `main` (`git status`: M world.json/README/PLAN + 2 untracked .import files). Commit/push before deploy or the change ships from nowhere.
- **Region banner overlaps quest log (portrait):** "The Reed Road" title renders across the quest-log's 4th line (`qa6_moved.png`, also in the pre-existing `game0.png`). Cosmetic, pre-existing, portrait only.
- **Canonical verify FAIL disposition — CONFIRMED SOUND:** `verify.mjs` exits 1 solely on "RULE LAYER NEVER RAN" because the generic harness can't type into the sign-in gate. Auth-aware runs prove the layer alive in real play: `start_amb`, `seen_gate` fired in normal wandering; `mp_respawn` fired on a driven death; `GOGI_RULES loaded vars=1 rules=10`. Harness limitation, not a game bug.
- **Audio:** infra present (verify: AudioManager/players/bus OK), `start_amb` ambient rule fired; actual playback unverifiable in the muted container.
- **Localhost-only placeholder noise:** serving out/ WITHOUT the CDN, shared props (`/godot-assets/props/...` torch/trees/bushes/rocks) 404 → gray placeholders. All 10 such paths return **200 from preview.myapping.com** (curl-verified) and render correctly when proxied (`qb1_game_prod_assets.png`: trees, bushes, logs, torch-in-hand). Environment artifact of local serving, not a build defect — listed so nobody re-flags it.

## ✅ Verified green (real deltas, real renders)
| Check | Evidence |
|---|---|
| Sign-in gate appears, correct copy | `qa2_gate.png` — "Sign in to play / saves your progress and plays with others", email+password, eye-toggle, Create one, Forgot password |
| Error states legible | Empty submit → "Enter a valid email address."; wrong password → red "Invalid login credentials" (`qa3_gate_error.png`); recovery to successful sign-in in same session |
| Auth passes via real UI | `GOGI_AUTH signed in uid=3f6dbd7c…` → `gate passed`; Supabase HTTP proxied through Node (container TLS block is env-only) |
| Session persistence | Page reload → `GOGI_AUTH restored existing session — no prompt` (no second login) |
| Multiplayer config live | `GOGI_MP enabled max_players=6`; `GOGI_MP resolved map=- room=cloud-iybqouv5yf6dymalatmc` (build-derived room under production path shape; empty-room in run 1 was my server serving at `/` — artifact) |
| Netsync doesn't degrade SP play | wss blocked in-container → only `[Net]` warnings; zero real console errors across both runs; movement/rules/quests all functional with netsync running |
| Realtime transport (from Node) | Re-ran nettest.mjs with world.json's exact URL+anon key: both clients SUBSCRIBED to `game:cloud-iybqouv5yf6dymalatmc-nettest`, A received B's broadcast — PASS |
| Model rebase (change 3) | All 5 refs `/cloud-iybqouv5yf6dymalatmc/models/meshy/*.glb` ↔ files in out/models/meshy/ match; wanderer + heron_villager fetched+rendered in-game; `GOGI_HERO native avatar attached (char_h=1.729)`; **zero GOGI_PLACEHOLDER / zero 404** in the production-shaped run |
| Meshy characters render (no gray boxes) | `qa7_npc.png` / `qa9_day.png`: textured yellow-coat wanderer (animating, feet grounded, `GOGI_HERO_SEAT 0.005`), dark heron NPC beside him |
| Movement + camera | Held W = 9.4 m real position delta; right-half drag = 1.33 rad cam_yaw delta (orbit works, no floor-stare) |
| NPC interaction | Walked to Pelli (12.8 m → 1.5 m), prompt "USE > Talk to Pelli", `try_use()` fired talk → 2 requests to npc.myapping.com (brain+speak) |
| Quest log + HUD | Village of Stilts 4-step log, minimap, region banner, PLACES SEEN counter (increments 0→1), JUMP/USE, HP/XP/Gold bar all present |
| mp_respawn well-formed | Static: `GOGI_RULES loaded rules=10`, rule fires on event, toast + respawn actions execute (the modal issue above is engine ordering, not rule syntax) |
| Winnability | Canonical verify: "quest-graph OK — world is winnable (project world.json, 64 areas)" — unchanged by delta |
| Night/day both readable | `qa8_night.png` (deep blue, boardwalk + hero readable, not black) / `qa9_day.png` (no blow-out; luma clipped=0.0% in verify log) — driven deterministically via gogiSetTime |
| Mobile fill | Portrait 390×844 + landscape 860×400 both fill all corners, HUD inside rect, no letterboxing (`qa10_landscape.png`) |
| Native tier | manifest.json: `webOnly: false`, world.json present — native-playable |
| Packaging / budget / scenes | Canonical: pck 5.1MB OK, scene pass 35/0, GPU peak 51MB/220MB budget, console clean |

## Could not verify (sandbox limits)
In-engine wss to Supabase from Chromium (container TLS block — transport proven from Node instead); real 2-device peer rendering (peer_body spawn/cull paths untested live); real audio playback; true-GPU fidelity; touch feel. World-boundary/streaming persistence not re-walked — pre-existing shipped content, unchanged by this data-only delta.

**Test account used:** the coordinator-supplied `verify-cloud-iybqouv5yf6dymalatmc@example.com` only; no new accounts created. My artifacts: `/workspace/verify/qa-probe.mjs`, `qa-probe2.mjs`, `qa*.png`, `qb*.png`. Repo untouched.
