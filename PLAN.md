# Goal
A peaceful exploration game: wander a small foggy fishing village built on stilts over a swamp.
Heron-people villagers (Meshy-generated) stand on the docks and can be talked to (LLM NPC chat);
one hut is enterable; a giant ancient catfish sleeps in the water beside the pier.
Built on the Godot 4.7.1 `godot-tmpl-rpg` data-driven base (native-tier: all content in
`world.json` + `quests.json`, no game-authored scripts), exported web/nothreads, Compatibility renderer.

# Files to touch
- `world.json` — chunk-mode swamp village (Architect specialist authors macro layout; coordinator
  integrates Meshy models, vars/rules/hud, director, audio hooks).
- `quests.json` — one auto-start quest: meet the elder → visit the pier & the catfish → step inside the hut.
- `structures` interior spec for the one enterable hut — Structures specialist.
- `models/` — Meshy characters (3 heron villagers, player wanderer, giant sleeping catfish).
- `audio/` — music + swamp ambient loops from the CC0 library.
- Starter template files committed as-is (engine scripts, export preset, project.godot).

# Verification approach
qgcheck winnability gate + `verify.mjs` smoke (boot, clean console, frames, pck/VRAM gates) +
targeted checks (facing, NPC chat contract + panel, trigger regions, mobile two-aspect fill) +
Game-Feel and QA specialist passes before the PR.

# Out of scope
Combat/enemies, multiplayer, accounts/backend (solo exploration — no persistence needed),
vehicles/mounts.
