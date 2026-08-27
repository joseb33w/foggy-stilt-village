# Goal
Turn Heronwade from a peaceful stroll into a bigger open world with combat and
rides: hostile characters to fight (three factions + a boss), a weapons
progression the player collects and uses, and a world large enough to drive a
car, pilot a boat, and fly a plane — all as world.json DATA on the existing
rpg engine (no game-authored scripts, native-tier preserved).

# Files to touch
- `world.json` — grid expanded 8×8 → 24×24 (x/z ∈ [-8..15], ~384 m across),
  existing 64 village cells preserved verbatim. New biomes: the Broad Murk bay
  (east, terrain carved below water level, boatable), the Reed Barrens + Old
  Causeway road grid (north, drivable), the Mirewood bog (west), Dragonfly
  Strip airstrip + the Reaver's Delta (south). New: `weapons` catalog +
  chest drops, per-cell enemy camps (`rat_bandit`, `pike_pirate`, `bog_lurker`,
  boss `murk_reaver` with Meshy models), world-level `vehicles[]` (car, boat,
  plane — Meshy multi-part bodies), new regions + vars/rules/hud (bounty,
  faction kill scoring), director boss bar + victory, combat HUD unhidden,
  `max_players` raised to 16 for the bigger map.
- `quests.json` — director chain extended: village_of_stilts →
  arm_the_village → clear_the_waters → rout_of_the_reeds (boss).
- `models/meshy/` — new rigged enemy GLBs + vehicle part GLBs (Meshy specialist).
- `README.md` — updated feature description.

# Verification approach
qgcheck (winnability incl. kill-count feasibility), canonical verify.mjs on the
export (packaging, GPU budget, rules-alive), targeted checks: combat delta
(enemy hp drops via real attack path), enemy AI engages, vehicle board/drive,
clip resolution on new enemy rigs, mobile two-aspect fill. Game-Feel + QA
specialist passes before ship.

# Out of scope
- PvP death scoring (`player_died` rules) — deliberately omitted: the engine
  fires its blocking defeat modal after the rule runs (documented engine
  follow-up from the previous session), so death keeps the engine's forgiving
  heal-in-place default.
- New enterable interiors (the elder's hut interior is preserved unchanged).
- Enemy in-hand weapon meshes — enemy armament is expressed as data
  (damage/range/speed profiles + faction weapon drops); the engine attaches
  weapon visuals to the player only.
