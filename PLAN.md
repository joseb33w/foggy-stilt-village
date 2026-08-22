# Goal
Add multiplayer to Heronwade so friends who open the same game link share the
village: see each other, walk together, and stay identifiable.

# Files touched
- `world.json` — added the `multiplayer` block (enabled, max_players 6, Supabase
  URL + publishable anon key; implies the engine sign-in gate). Rebased the five
  Meshy model refs from the stale previous-build prefix to the committed
  `models/meshy/` paths (fixes asset 404s). Made the title backdrop opaque
  (`director.title.bg` alpha 0.94 → 1.0) so the sign-in form no longer ghosts
  through the title screen.
- `auth_gate.gd` — scale-aware touch metrics: the project stretches a 720-wide
  base viewport onto ~390pt phones, so the gate's "44pt" fields rendered ~24pt.
  Sizes/fonts now multiply by canvas-units-per-screen-point (clamped ≥1 so
  desktop is unchanged), restoring real 44pt+ tap targets.
- `README.md` — documents the multiplayer behavior and private rooms.

Decision of note: a `player_died → toast + respawn` rule was added, then REMOVED
after both QA and Game-Feel specialists proved (empirically) that the engine
fires `_show_defeat()` AFTER the rule runs — a blocking DEFEATED/TRY AGAIN modal
lands on the already-respawned player, and TRY AGAIN does a full run reset. An
engine patch here would fix only the web build while native players (who run the
canonical engine) still got the broken modal, so the world does not opt into
death: the engine's seamless heal-in-place default applies. Engine-side call
ordering is reported as a follow-up.

# Verification
- qgcheck winnable; canonical verify.mjs green on packaging (5.1MB pck), scene
  instantiation 35/0, GPU 51/220MB, zero 404s/placeholders, console clean. Its
  one FAIL ("RULE LAYER NEVER RAN") is a known harness limitation — the generic
  harness cannot type into the sign-in gate; an auth-aware probe
  (verify/auth-probe.mjs technique) signed in through the real UI and proved the
  same signal green (rules fired, movement, camera, netsync engaged
  max_players=6). QA independently confirmed this disposition sound.
- 2-client Node Realtime transport test: both clients subscribed to the engine's
  topic shape with world.json's exact creds; A received B's broadcast.
- Independent QA + Game-Feel specialist passes (reports in docs/); all P0/P1
  findings fixed and re-proven before ship.

# Out of scope
- PvP scoring / deathmatch rules (co-op stroll; the only weapon is a 1-damage
  reed torch).
- Shared/host-authoritative NPCs or quest state (engine syncs position, facing,
  shots and hits; quest progress stays per-player).
- Leaderboards or any new backend tables (Realtime broadcast needs none).
