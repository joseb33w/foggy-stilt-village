# QA Report — Heronwade "Village on Stilts" · Mire Dragon session (CONTINUE build)

**Build:** repo `/workspace/repo` (branch `main`, delta uncommitted), export `/workspace/repo/out`,
BUILD_ID `cloud-hdro8hilebm5ifgmt2hy` · world.json md5 `e5e2d56faf67d4aaf6c0ada4fa47940c` (final state verified)

## VERDICT: PASS (0 P0) — with 1 P1 and polish notes below

⚠️ **IMPORTANT CONTEXT — the build changed mid-QA.** The dragon I was delegated to review
(aerial: `enemy_aerial:true, hover 5.0, height 4.5, range 6.0`) was **rewritten by you at ~03:39
while my probes ran** into a **grounded** boss (`enemy_height: 2.2`, no aerial/hover/range keys).
I verified BOTH revisions. That change was the right call — my probes had independently proven the
aerial revision **unwinnable in melee** (details in §2). Everything below headed "CURRENT" is
verified against the final bytes (md5 above); the export `out/` world.json is identical to the repo
world.json, and `index.pck` is unchanged since 02:35 (md5-matched across all my probe copies).

---

## 1. What passes (CURRENT build, real evidence)

| Check | Result | Evidence |
|---|---|---|
| verify.mjs full gate (mp-off byte-variant, local engine) | ✅ `=== VERIFY PASSED ===`, console clean, exit green | `/workspace/qa/logs/verifyG.log` (current bytes), `verifyA.log` (earlier rev) |
| qgcheck winnability (campaign regression) | ✅ "world is winnable (576 areas)"; quests.json untouched (git-clean) | verifyG.log |
| **Dragon kill chain at REAL ship stats (hp 700), melee, real `_attack()` path** | ✅ hp fell 700→dead in 39 hits × 18 dmg over ~40 s; `dragon_aggro` → `dragon_enrage` (0.5 crossing) → `bounty_dragon` (kill) all fired | `/workspace/qa/logs/native6.log` (QA_DIAG hp ladder 700/682/…/-2 + GOGI_RULE_FIRED lines) |
| Dragon engages + damages the player | ✅ grounded chase closes to 1.6 m, player hp bar drops, `player_damaged`→`dragon_aggro` fires; feedback is the non-modal red flash + shake (no popup) | native6.log, `/workspace/qa/cur_t2.png`, `cur_t3.png` |
| Hoard chest → Dragonfang + 200 gold → auto-equip | ✅ `QA_EQUIP wpn=dragonfang inv=[rusty_sword, …, dragonfang] gold=200` via real `try_use()`/chest path; "USE > Open Chest" prompt + "You opened the chest." in-browser | `/workspace/qa/logs/native3.log`, `/workspace/qa/kill_chest.png`, `kill_weapon.png` |
| Ranged also damages/kills the dragon (weapon-curve sanity) | ✅ bow kill chain proven (hp 90 variant): qa_hit + enrage + bounty fired via projectile `take_hit` | `/workspace/qa/logs/native2.log` |
| Dragon model (Meshy rig-lab) | ✅ 8.8k tris, webp textures, 15 clips (attack/bite/death/…/walk); three DISTINCT rendered poses (idle/flap/attack) — skeleton moves, no T-pose; on-style mossy swamp dragon | `/tmp/qa_render/dragon_{idle,flap,attack}.png`, meshy_assets.jsonl entry (match PASS) |
| Dragon scale in-scene | ✅ authored 2.2 m tall → ~4.6 m long body + wide wingspan; reads boss-sized next to the 1.65 m hero, not comical, not frame-filling | `/workspace/qa/cur_t2.png`, `bow360_h5.png` |
| Roost dressing + region rules | ✅ dirt lair w/ bones/skull/ribcage/dead trees/rocks renders; "The Dragon's Roost" region title, tension-music region entry, `seen_12` (+explored, max 13 ✓), `roost_warn` subtitle+shake, `roost_rumor` authored on Mirewood | `cur_t2.png`, GOGI_RULE_FIRED lines in every probe log |
| Melee vs ground enemies (regression) | ✅ 3 rat bandits killed cleanly, 18 dmg/swing, 70→-2 each, through the same `_attack()` | `/workspace/qa/logs/native5.log` |
| Mobile fill | ✅ portrait 400×860 and landscape 860×400 both fill all four corners, HUD inside viewport, no overlap, title screen fills too | `/workspace/qa/fill_portrait.png`, `fill_landscape.png` |
| Day / night readability | ✅ deterministic gogiSetTime capture: day mean 145.9 (0% clipped), night mean 38.1 — night stays readable, day doesn't blow out | verifyG.log luma lines, `/workspace/verify/luma-{day,night}.png` |
| Boot + console | ✅ engine boots, canvas present, "console clean (no real JS/GDScript errors)"; zero SCRIPT ERROR / rules-UNIMPL lines across ~10 gameplay sessions | verifyG.log + all probe logs (`ERRS: none`) |
| Native tier | ✅ chunk-mode world.json; manifest `webOnly:false`, requires rules+hud | `/workspace/repo/out/manifest.json` |
| Character sourcing (Meshy mandate) | ✅ hero/NPCs/enemies/dragon all Meshy GLBs (`models/meshy/*`); mire_dragon staged locally AND live on the CDN (200, 2,492,644 bytes) | manifest + curl check |
| Audio | ✅ AudioManager + bus layout present; `thunder` used by dragon rules is a registered SFX; region music `tension.ogg` shipped | verifyG.log "audio infra present", `out/audio/` |
| World persistence / containment | ✅ terrain border walls + always-on `_clamp_to_world`; feel streaming probe walked cells with no fall-through; cross-map teleport rebuilt cells and enemies correctly | verifyG.log, native5.log |

## 2. The P1 — resolved mid-session by your grounding change, but VERIFY MY CLAIM & finish the loose ends

**The DELEGATED (aerial) dragon was melee-unwinnable and half the time never attacked.** Proof
gathered before your 03:39 edit:
- Native instrumented runs: the flyer's keep-away hard-clamped horizontal distance to a constant
  **3.89 m / 5.52 m / 8.58 m** (per-spawn `_air_keep` draw ∈ [0.75,1.7] × surround 5.1) — always
  above the 2.4 m melee reach; swoops dip only VERTICALLY (`enemy.gd` swoop lerps y, horizontal
  target stays the keep-distance slot). hp stayed 90 under continuous real-path attacks
  (`/workspace/qa/logs/native4.log` hdist=8.58 forever, no aggro — the >6 m draw also never attacks).
- Browser sessions: ~360 ATTACK swings across 2 engaged sessions + a 360°×24-arrow bow sweep — zero
  `enemy_hp_crossed` events (`melee.log`, `bow360.log`).
- The engine comment says the swoop exists so the flyer is "reachable mid-dive instead of parked
  untouchably overhead" — the keep-away contradicts it. **Latent engine bug class for ANY future
  aerial melee-range enemy** (enemy.gd keep-away vs melee reach); this game no longer ships an aerial
  enemy, so it's not a blocker here, but worth an engine note.

**Loose ends from the grounding change (do these):**
1. **Stale docs**: PLAN.md still says "dragon flies (aerial + hover, swoops to attack)" and the
   README/delegation describe an aerial boss. The shipped dragon WALKS. Align the docs (or restore
   flight only if the engine keep-away/melee interaction is actually fixed).
2. `enemy_range` was dropped (now default 2.0) — verified fine (it lands hits), just noting the
   delta from the described design (6.0).

## 3. Polish / P2 notes (non-blocking)

- **Hoard chest is lootable while the dragon is alive** — I opened it mid-fight and took
  Dragonfang + 200 gold with the boss at full hp (`native3.log`: chest opened, aggro fired after).
  The bounty toast says "Its hoard is unguarded" implying kill-gating. Consider `"lock"`-ing the
  chest or gating on the kill flag if guarding is intended.
- **Missing collider**: verify feel probe ended INSIDE a solid AABB near [-24.3, 30.7] (wilderness
  cell (-2,1)/(-2,2) scatter — a tree/log/stump visual without a matching collider). Reproduced in
  both verify runs. world-streaming SOLID-BY-DEFAULT says fix it.
- **Fresh-cell hitch**: worst frame 483–500 ms while walking into new cells (both verify runs).
  Real device stall (not container noise) — find the synchronous work in the cell build and slice it.
- **SPARSE lint** (~1.9 weighted content/cell) + flat-tint lint: pre-existing, not this session's
  delta; visuals judged acceptable for a fog-bog wilderness (village/camps read authored; characters
  are demonstrably textured in every screenshot). No action demanded.
- **Branch bookkeeping**: delegation said branch `feat/mire-dragon`; the repo is on `main` with the
  whole delta UNCOMMITTED (and you were editing world.json during QA). Commit before anything else
  touches the tree.

## 4. Could not verify (sandbox limits)

- Real multiplayer/auth (supabase TLS unreachable from container Chromium) — per the known
  constraint, all gameplay probes used byte-identical `multiplayer.enabled=false` variants + local
  engine-html patch; the canonical export's only verify FAIL remains the auth-gate "RULE LAYER NEVER
  RAN" artifact. Second-device sync, host election, and the live wss path are untested here.
- Real-GPU fidelity, audio playback, touch feel.

## 5. Method note

All variants/instrumentation lived in **my own copies** (`/workspace/qa/out*`, `/tmp/qa_proj`);
`/workspace/repo` and `/workspace/repo/out` are untouched (git status shows only your session delta).
Evidence: screenshots in `/workspace/qa/*.png`, `/tmp/qa_render/dragon_*.png`; logs in
`/workspace/qa/logs/`.
