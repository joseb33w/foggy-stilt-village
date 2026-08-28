# Goal
Add a dragon to Heronwade: the **Mire Dragon**, an aerial boss creature roosting
in a new bone-strewn lair in the far south-west bog. Pure world.json DATA on the
existing rpg engine (no game-authored scripts — native tier preserved): the
dragon flies (aerial + hover, swoops to attack), guards a hoard chest with a new
top-tier weapon, and is wired into the rumor/bounty/discovery rule layer.

# Files to touch
- `world.json` —
  - new region **The Dragon's Roost** (cell [-7,10], world ~[-104,168], music: tension),
  - roost cells dressed with bones / skulls / dead trees / rock scatter,
  - lair cell: 1 aerial `mire_dragon` (hp 700, dmg 24, height 4.5, hover 5,
    Meshy model `models/meshy/mire_dragon.glb`), hoard chest (Dragonfang + 200 gold),
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
critique of the roost, targeted checks: dragon spawns + hovers aerial, flap clip
plays, kill → bounty rule fires, chest grants Dragonfang. QA + Game-Feel
specialist passes before ship.

# Out of scope
- No changes to the campaign chain / goal (`rout_of_the_reeds` untouched — the
  dragon is a side encounter; rules-layer only, no quests.json edit since the
  quest chain is director-managed).
- No rideable dragon mount (the request says "add a dragon in the game"; the
  hostile roost encounter is the chosen reading — noted in PR body).
