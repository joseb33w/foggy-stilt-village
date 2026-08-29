# Goal
Add a second dragon to The Dragon's Roost: a three-headed hydra dragon in blue-mixed-with-red scales (user request), Meshy-generated with the same G_DRAGON rig-lab clip set as the existing Mire Dragon.

# Files to touch
- `world.json` — dress cell [-6,10] (adjacent to the Mire Dragon's lair, inside The Dragon's Roost) as a second bone-strewn lair with the `hydra_dragon` enemy (hp 900, dmg 24, height 2.5) + a gold hoard chest; update the roost rumor/warn flavor; add `hydra_enrage` + `bounty_hydra` rules (350 bounty / 350 xp).
- `models/meshy/hydra_dragon.glb` + `models/meshy_assets.jsonl` — new Meshy creature (delegated to the Meshy specialist).
- `README.md` — Roost description, quests paragraph, credits line, play link.
- `main.gd` — engine re-sync (template update, no game changes).

# Verification approach
Re-export the Web (nothreads) build, run the canonical verify.mjs smoke harness, targeted check that the hydra cell streams its enemy with the new model (three heads + blue/red visible in a render), QA specialist final pass, deploy preview, PR + merge.

# Out of scope
No quest-chain changes (the dragons stay optional side encounters), no new weapons/items, no layout changes elsewhere in the world.
