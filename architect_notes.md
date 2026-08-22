# Architect notes — "Heronwade — Village on Stilts"

Macro-layout for the foggy heron-fisherfolk stilt-village request. Generated deterministically by
`/tmp/gen_world.py` (left outside /workspace per the write-only-world.json contract; re-run it with
`python3 /tmp/gen_world.py` to regenerate after tweaks).

## Shape of the world
- **Chunk mode, cell_size 16, 8×8 grid (gx/gz −1..6), 64/64 cells populated → 128×128 m, fully contiguous.**
  Core village = 6×6 (gx/gz 0..5, ~96 m across); the outer ring is mud/reed swamp edge (dense reed/bush/dead-tree
  scatter + driftwood props — atmosphere, not void).
- **Sky:** `{"time":"sunset","weather":"fog"}` — amber-grey fog, per request.
- **Terrain:** `mud`, amplitude 2.2, freq 0.025, floor **+0.8** — the village ground/banks sit ~0.5–2.4 m above
  water; noise occasionally dips below for natural puddles. Authored `features` guarantee the water bodies:
  - a **canyon channel** along the east edge (world x≈96, full N–S, width 10, depth 4) → the wide swamp channel
    the pier overlooks;
  - four **basin pools** at (24,72), (−12,12), (56,−12), (12,−12) — placed in gaps, ≥11 m from any structure/walkway.
- **Water:** level −0.3, murky olive shallow [0.32,0.35,0.21] → brown-green deep [0.09,0.12,0.07], wave_amp 0.06.

## Circulation (no roads — boardwalks)
- EW boardwalk spine on row gz=2: reed-road entry plank on c-1_2 → gate c0_2 → square c2_2 → pier c5_2.
- NS boardwalk on column gx=2 (local x=3): north jetty stub c2_0 → square → south fishing deck c2_4 (which
  overlooks the (24,72) pool).
- Every walkway = plank `rows` (thin timber boxes, no collider — walk-over decoration) flanked by thin timber
  post `rows` (cylinder, r 0.15, h 2.5 — solid, spaced 2.7 so passage is never blocked).

## Structures — 11 timber/thatch gable huts + 1 watch-post (all parametric, zero GLB)
- Varied footprints 5×5…7×6, yaws ±5–20°, warm `window_glow` [1.0,0.72,0.42]; two are `floors: 2`
  (c1_3, c4_1) for skyline variation. Each hut gets a stilt-post fringe row + lantern/barrel/crate dressing.
- **THE enterable hut (exactly one):** on the square, **c2_2 pos [−3.5,−4.5], footprint 9×7**,
  `interior: {door:"hinged", door_face:"s", rooms:1, lit:true}` — door opens south onto the EW boardwalk.
  Structures specialist refines the interior later; the footprint + door face are reserved and clear.
- **Landmark:** leaning timber watch-post (taper, h 7.5) on c4_2 at the pier approach.
- **The great pier (c5_2):** plank approach + 3-row pier-head platform (~5×7.5 m) at the channel lip
  (planks end ~10 m short of the channel centerline → stay on dry bank), edge posts, 4 lantern torches,
  crates/barrels. Social hub: fisher NPC + a wander crowd.
- No overlapping footprints (checked programmatically); props/NPCs/chest audited against footprints + walkways.

## Cast (all `model`-less — coordinator wires Meshy paths; `default_npc_model` left out per contract)
| id | cell | pos (local) | role |
|---|---|---|---|
| `pelli` | c0_2 | [2.5,3.2] | young heron at the western gate |
| `elder_sedge` | c2_2 | [−3.5,2.6] | elder outside the enterable hut (quest: talk-to target) |
| `netmender_moorwick` | c3_2 | [2.0,−3.5] | net-mender by the drying-rack rows |
| `fisher_skerrin` | c5_2 | [4.8,0.5] | fisher on the pier head |
| `old_catfish` ("Old Murk") | **c6_2** | [−2,0] → world (94,32) | colossal sleeping catfish IN the channel beside the pier — terrain there is ≥0.9 m below water by construction (canyon), so a big model reads as dozing in the shallows under the pier |

`populate`: 2 entries (c2_2 square + c5_2 pier), each `{"set":["MESHY_VILLAGER_SET"],"count":3,"vary":true,
"behaviour":"wander","radius":6,"speed":1.1}` — substitute the placeholder string with real staged paths.

## Gameplay graph
- `start_cell: [0,2]` — gate cell, spawn at cell centre on clear mud (planks are non-colliding; no structure on
  the cell). First view: gate posts + lanterns, boardwalk leading east to the lit square.
- `goal: {"type":"complete_quest","target":"village_of_stilts"}` — **coordinator must author quests.json with
  that quest id** (suggested steps: `talk_to` → `elder_sedge`; reach the pier — `reach_area` target `"c5_2"` or
  region `the_pier`; visit Old Murk / step inside the elder's hut — region `elders_hut`). qgcheck will FAIL
  until quests.json carries the id — that's the expected ordering, not a layout bug.
- `items`: `heron_charm` (non-consumed token) in a chest INSIDE the enterable hut at c2_2 local [−6.5,−6.5]
  (clear of the door swing) + 10 gold — flavour reward for stepping inside.
- **Zero enemies, zero locks** — peaceful exploration; every cell is open-adjacency, goal trivially reachable.
- `regions` (ambience toasts): `village_gate` (0,32 r10) · `village_square` (32,32 r12) · `the_pier` (84,32 r10)
  · `catfish_pool` (94,32 r8) · `elders_hut` (28.5,27.5 r6).

## Honest caveats
- Terrain noise is stochastic around the authored features: with seed 7 the banks/pools read right by
  construction margins (~0.5 m+), but if a rendered frame shows a stray puddle lapping a hut base, nudge
  `terrain.seed` — the authored canyon/basins are seed-independent.
- The pier is a bank-edge dock (planks ground onto terrain), not a cantilevered deck over open water — the
  engine grounds row parts, so the "over water" read comes from the pier head sitting at the channel lip with
  Old Murk in the water directly beyond. If the coordinator wants planks physically over the water surface,
  that needs an engine-side elevated-deck primitive (out of layout scope).
- Budgets: max 8 props/cell (cap 12), scatter ≤18/entry (cap 40), rows ≤13 parts (cap 80), 12 structures total,
  1 interior (budget: 2–3× nodes, only one used). No GLB references at all — model-memory cost is zero until
  Meshy assets are wired.
