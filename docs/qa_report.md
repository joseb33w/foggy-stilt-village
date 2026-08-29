# QA Report — Heronwade · Hydra Dragon session (CONTINUE build, final content-complete pass)

**Build:** repo `/workspace/repo` (on `main`, session delta uncommitted; `models/meshy/hydra_dragon.glb` untracked),
export `/workspace/repo/out` — world.json md5 `4ea93a340b37adb9c4a91450939e692c` (repo == out, byte-identical),
quests.json untouched (`5218eec2…`, git-clean), pck/wasm 03:19 > main.gd 03:13, manifest 29 files incl.
`models/meshy/hydra_dragon.glb`, `webOnly:false`. All probes ran against MY OWN copies
(`/workspace/qa/out-mpoff`, `/workspace/qa/out-visual`, `/workspace/qa/proj`) — repo + out untouched
(git status shows only your session delta).

## VERDICT: PASS (0 P0, 0 new P1)

---

## 1. This session's delta — all verified with real deltas

| Check | Result | Evidence |
|---|---|---|
| **Hydra kill chain at REAL ship stats (hp 900), real `_attack()` path** | ✅ hp 900 → 0 over 81 swings × 18 dmg (Reed Torch/rusty_sword); hp ladder logged 900/882/…/0 | `/workspace/qa/logs/native2.log` QA_DIAG lines |
| **`hydra_enrage` fires at the 0.5 crossing** | ✅ `GOGI_RULE_FIRED hydra_enrage on=enemy_hp_crossed` between hp 522 → 432 (crossing = 450) — exact threshold | native2.log |
| **`bounty_hydra` fires on kill (+350 bounty, +350 xp)** | ✅ `GOGI_RULE_FIRED bounty_hydra on=kill`; bounty var 0 → 350; xp 0 → 41 **after 5 level-ups** — 350 (rule) + 15 (engine kill xp) = 365; xp_next chain 30+42+58+81+113 = 324; 365−324 = 41, matches EXACTLY | native2.log QA_AFTER_KILL |
| **Player takes 24/hit from the hydra, real damage path** | ✅ hp 100.0 → 76.0, delta = 24.0 exactly (authored enemy_damage); `dragon_aggro on=player_damaged` fired; feedback is the non-modal red flash (no popup) | native2.log QA_PLAYER_DAMAGED; browser run repro'd dragon_aggro |
| **New lair chest opens (+250 gold, gold-only)** | ✅ gold 0 → 250 via real `interaction.try_use()` → `_open_chest` | native2.log QA_CHEST |
| **Model: three heads, blue + red, textured** | ✅ THREE distinct horned heads on separate necks; cobalt-blue scales, crimson mottling, red wing membranes w/ blue veining, pale underbelly, ember eyes — fully textured (3 webp), NOT gray/flat-tinted | `/workspace/qa/shots/glb_front_idle.png`, `glb_side_idle.png` (look at them) |
| **Animated, no frozen T-pose** | ✅ 15 clips incl. idle/walk/attack/death (GLB parse); idle vs attack vs walk are visibly different poses; max bone-position delta idle→attack 0.163, idle→walk 0.142 (nonzero = skeleton moves) | `glb_attack_34.png`, `glb_walk_side.png`, `glb_death_34.png`; qa_glb_render.mjs output |
| **Scale + facing in-scene** | ✅ rendered AABB in-game 3.15 × **2.50** × 3.66 m — exactly the authored enemy_height 2.5 cap; reads boss-sized beside the 1.65 m hero, not frame-filling; faces the player while attacking (no moonwalk) | native2.log QA_HYDRA_AABB; `/workspace/qa/shots/hy_d.png` |
| **Lair cell [-6,10] dressing renders** | ✅ dirt ground, bones, skull, ribcage, dead trees, rocks, gold chest all visible in-browser; every lair prop fetched (ribcage/skull/tree_dead_large/CommonTree_Dead_1 + hydra glb) | `hy_b.png`–`hy_d.png`; probe fetch list |
| **No spawn-inside-collider** | ✅ hydra spawned at (−84.4, 0.4, 168), chased + attacked immediately (not stuck); player spawn/movement in the lair clean; chest reachable | native2.log; browser probe |
| **Both dragons coexist, neither breaks the other** | ✅ mire_dragon converged into the hydra fight (both chase from spawn — matches the authored "Two sets of wingbeats" fiction); each tracked its OWN hp; `dragon_enrage` fired on the MIRE's 0.5 crossing while I fought the hydra, `hydra_enrage` on the hydra's — target discrimination correct both ways | native2.log rule-fire ordering |
| **Mire dragon regression (kill chain)** | ✅ killed through the same path; `GOGI_RULE_FIRED bounty_dragon on=kill`; bounty 350 → 650 (+300), xp granted; hp 700 pool + 20 dmg/hit confirmed at spawn readback | native2.log QA_MIRE_* |
| **Reworded flavor (roost_rumor / roost_warn / dragon_aggro)** | ✅ diff-verified ("TWO dragons… one has three heads"; "Two sets of wingbeats"; aggro no longer names the Mire Dragon); `roost_warn` + `dragon_aggro` proven live this session (rumor is text-only reword on a previously-proven trigger) | git diff; native2.log / browser log |

## 2. Standing invariants (CONTINUE rule) — re-checked on the whole game

| Check | Result | Evidence |
|---|---|---|
| Canonical verify.mjs, mp-off byte-variant | ✅ `=== VERIFY PASSED ===`; engine 70/70 current; scene pass 36/0; console clean; spawn clear; streaming OK (18.4 m, no fall-through); world responds to input | `/workspace/qa/logs/verify_mpoff.log` |
| qgcheck winnability | ✅ "world is winnable (576 areas)" — campaign quests untouched, dragons stay optional | verify_mpoff.log |
| Rule layer alive | ✅ start_amb/seen_12/roost_warn/dragon_aggro + all 4 new-delta rules fired across sessions; 29 unfired rules are kill/region/once-gated (expected) | logs |
| Day/night luma | ✅ day mean 163.3, 0% clipped; night 47.7 (readable) | verify_mpoff.log luma lines |
| Camera orbit + HUD | ✅ cam_yaw 0 → 4.21 on right-half drag; HUD (quest panel, bounty, buttons, minimap) inside 860×400 frame, no overlap | browser probe player t1/t2; hy_c/hy_d |
| Audio infra | ✅ "audio infra present"; `thunder` SFX used by hydra_enrage is the registered SFX the mire rules already use | verify_mpoff.log |
| Meshy mandate | ✅ hydra is Meshy rig-lab G_DRAGON (meshy_assets.jsonl match:PASS, 9,114 tris, 3 webp textures, 15 clips); all characters remain `models/meshy/*` | jsonl + GLB parse |
| Export/native tier | ✅ chunk-mode world.json copied loose; manifest `webOnly:false`, requires rules+hud; hydra glb md5 identical repo↔out | out/manifest.json, md5s |

## 3. Notes / polish (no new defects; pre-existing P2s re-observed, do NOT re-fix blindly)

- **Pre-existing P2s persist unchanged** (flagged last session, not this delta): fresh-cell hitch — worst
  frame **467 ms** this run (was 483–500); SPARSE lint ~1.9/cell; flat-tint static lint; the missing-collider
  report near [−24.3, 30.7] was not re-triggered this run (FEEL collision OK) but was not specifically re-probed.
- **New hydra hoard chest is lootable while the hydra lives** — same ungated-`chest` class as the mire hoard
  P2 from last session (no `lock`/flag gate in the cell JSON). Same severity, same optional fix.
- **Stale build-id model URLs** — mire (`/cloud-hdro8hilebm5ifgmt2hy/…`) and bog_lurker
  (`/cloud-cimwbwzhmp76phh40et8/…`) ride main.gd `_norm()`'s build-id self-heal (proven: both glbs fetched +
  rendered in my probes). Zero action required; authoring new cells with the current id (as the hydra does) stays tidier.
- **Advisory FEEL ascent WARN** (traversed 5.4 m, height changed <0.5 m) — sampling-position advisory on
  rolling amplitude-2.2 terrain; streaming/collision probes passed; not observed as a real traversal failure.
- **Branch bookkeeping (again):** delegation names `feat/hydra-dragon`; the repo is on `main` with the whole
  delta uncommitted and the glb untracked. Also three stray verify screenshots (`frame0..2.png`, 03:50,
  untracked — from a verify run launched with the repo as cwd, before this QA pass) sit in the repo root;
  delete them before committing. Same commit-hygiene note as last session.
- **Preview URL not live yet** at QA time (`https://preview.myapping.com/cloud-nusrgc4jlryff4h2e2cr/` → 404) —
  all verification ran against the local canonical export per the delegation; spot-check the URL after deploy.

## 4. Could not verify (sandbox limits)

- Live multiplayer/auth (supabase TLS unreachable from container Chromium) — canonical export still parks at
  the auth gate here; every gameplay probe used byte-identical `multiplayer.enabled=false` copies. Second-device
  sync and host election untested.
- The deployed preview URL (not yet live), real-GPU fidelity, audio playback, touch feel.

## 5. Method

Native instrumented runs: my own project copy (`/workspace/qa/proj`) + a QA autoload driving the REAL
`main._attack()` / `interaction.try_use()` / enemy-attack paths against a locally-served mp-off world
(props proxied from the live CDN). Browser probe: Chromium/SwiftShader on a start-at-lair variant, real
SET OUT click, real orbit drag. Model render: three.js/SwiftShader clip-posed captures of the shipped GLB.
Logs: `/workspace/qa/logs/`; screenshots: `/workspace/qa/shots/` (all judged by eye).
