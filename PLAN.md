# Goal
Add multiplayer to Heronwade so friends who open the same game link share the
village: see each other, walk together, and stay identifiable.

# Files to touch
- `world.json` — add the `multiplayer` block (enabled, max_players 6, Supabase
  URL + publishable anon key; implies engine sign-in) and an `mp_respawn` rule
  (`player_died` → toast + respawn) so a downed player rejoins instead of
  silently auto-healing.
- `README.md` — document the multiplayer behavior and private rooms.

No engine/script changes: the template already carries the netcode (`net.gd`,
`netsync.gd`, `peer_body.gd`) and auth (`auth.gd`, `auth_gate.gd`); the world
turns them on with data, which keeps the build native-player streamable.

# Verification approach
- qgcheck winnability gate + full `verify.mjs` smoke pass on the re-export
  (includes the multiplayer map-spacing check: 8×8 cells @ 16 m ≈ 52 m mean
  spacing at 6 players — inside the 25–120 m band).
- 2-client Node transport test against the real Supabase Realtime endpoint
  (two channels on one room topic; one broadcasts, the other receives).
- Independent QA + Game-Feel specialist passes on the finished export.

# Out of scope
- PvP scoring / deathmatch rules (this is a co-op stroll; the only weapon is a
  1-damage reed torch).
- Shared/host-authoritative NPCs or quest state (engine syncs position, facing,
  shots and hits; quest progress stays per-player).
- Leaderboards or any new backend tables (Realtime broadcast needs none).
