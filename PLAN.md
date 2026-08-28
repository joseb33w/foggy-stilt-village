# Goal
Add a dragon to Heronwade: the **Mire Dragon**, a hulking boss creature roosting
in a new bone-strewn lair in the far south-west bog. Pure world.json DATA on the
existing rpg engine (no game-authored scripts — native tier preserved): the
dragon stalks its roost, guards a hoard chest with a new
top-tier weapon, and is wired into the rumor/bounty/discovery rule layer.
(Originally authored as an aerial swooper; grounded after the Game-Feel + QA
passes proved the engine's flyer stand-off ring makes melee engagement a
spawn-time coin flip — see PR body.)

# Files to touch
- `world.json` —
  - new region **The Dragon's Roost** (cell [-7,10], world ~[-104,168], music: tension),
  - roost cells dressed with bones / skulls / dead trees / rock scatter,
  - lair cell: 1 `mire_dragon` boss (hp 700, dmg 20, height 2.2 -> ~4.6 m body,
    grounded stalker — the aerial profile's stand-off ring proved a spawn-time
    coin flip vs its own attack reach, per the Game-Feel pass; Meshy model
    `models/meshy/mire_dragon.glb`), hoard chest (Dragonfang + 200 gold),
  - `weapons` += **Dragonfang** (melee, dmg 72 — new top of the curve),
  - rules: Mirewood rumor toast, roost-entry warning subtitle + shake, aggro
    stinger (`enemy_detected_player`), half-HP enrage beat (`enemy_hp_crossed`),
    kill payoff (bounty +300, xp +300, toast), discovery counter (explored max 12→13).
- `models/meshy/mire_dragon.glb` — new Meshy-generated rigged swamp dragon
  (G_DRAGON rig-lab clips idle/walk/flap/glide), decimated for mobile;
  `models/meshy_assets.jsonl` metadata line appended.
- `README.md` — world/weapons/quest sections updated for the new encounter.

# Verification approach
qgcheck (graph unchanged — no new locks; must stay PASS), canonical verify.mjs
on the export (packaging, GPU budget, rules-alive, character tri gate), frame
critique of the roost, targeted checks: dragon spawns + engages, clips
play, kill → bounty rule fires, chest grants Dragonfang. QA + Game-Feel
specialist passes before ship.

# Out of scope
- No changes to the campaign chain / goal (`rout_of_the_reeds` untouched — the
  dragon is a side encounter; rules-layer only, no quests.json edit since the
  quest chain is director-managed).
- No rideable dragon mount (the request says "add a dragon in the game"; the
  hostile roost encounter is the chosen reading — noted in PR body).
