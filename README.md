# HERONWADE — a foggy village on stilts

An open-world adventure for the phone browser. Wander a fishing village built on
stilts over a swamp, talk to the tall heron folk — then head out: reed bandits
prowl the Old Causeway to the north, bog lurkers haunt the Mirewood to the west,
pike pirates hold an anchorage across the Broad Murk bay, and the skull-masked
Murk Reaver waits on a delta island. Drive the marsh buggy up the causeway, sail
a skiff across the bay, or fly the Dragonfly seaplane from the southern strip.

**Play:** https://preview.myapping.com/cloud-cimwbwzhmp76phh40et8/index.html

## How to play
- **Move:** left-side touch joystick (or WASD)
- **Look:** drag the right side of the screen (or mouse)
- **Talk / open / board rides:** USE button
- **Fight:** ATTACK button — weapons you find auto-equip when stronger; the
  WEAPON button cycles your arsenal
- **Jump:** JUMP button (or Space)

## The world
A 24×24-cell chunk-streamed open world (~384 m across):
- **Heronwade village** (center-west) — the original stilt village, boardwalks,
  the elder's enterable hut, Old Murk asleep under the great pier.
- **The Reed Barrens & Old Causeway** (north) — a drivable road grid, two
  rat-bandit camps, and the Marsh Buggy parked by Tinker Rasp.
- **The Mirewood** (west) — a dark bog of dead trees and bog lurkers, with a
  ruined watchtower.
- **The Broad Murk** (east) — open water for the Mud Skiff and Reed Runner
  boats; pike pirates hold the Anchorage island.
- **Dragonfly Strip** (south) — an airstrip with the Dragonfly seaplane.
- **The Reaver's Delta** (south-east) — the boss island, reached by boat or plane.
- **The Dragon's Roost** (far south-west) — a bone-strewn lair in the bog where
  the **Mire Dragon** circles overhead: an aerial boss that swoops to attack and
  guards a hoard chest holding the Dragonfang.

## Weapons
A seven-piece arsenal, all data-driven: the Reed Torch (starter), Fisher's Gaff,
Bog-Oak Bow, Marsh-Lantern Staff, Pike Harpoon, the Heron Talon Blade
(gifted for clearing the waters), and the Dragonfang (in the Mire Dragon's hoard).
Chests at each camp carry the local faction's weapon; stronger finds auto-equip.

## Quests
The chain runs *Village of Stilts* → *Bandits on the Causeway* → *Mire and
Murk* → *Rout of the Reeds* (defeat the Murk Reaver). The Mire Dragon is an
optional side encounter outside the chain — a rumor toasts in the Mirewood, and
felling it pays 300 bounty and 300 xp. Thirteen named places toast as you
discover them; a BOUNTY counter scores your kills.

## Multiplayer
The world is shared: everyone who opens the same game link lands in the same
room (up to 16 players) — position, facing, and hits sync over Supabase
Realtime. Sign-in (email + password) is required so friends are identifiable;
the session is remembered across launches. For a private room, add
`?room=<code>` to the link and share it.

## Tech
- **Godot 4.7.1**, Compatibility (WebGL2) renderer, single-threaded (`nothreads`)
  web export — runs in Safari/Chrome/Firefox on mobile and desktop.
- Data-driven world on the `godot-tmpl-rpg` engine: everything that defines the
  place lives in `world.json` (chunk-streamed grid, terrain + swamp water + fog
  weather, parametric structures, weapons catalog, enemy camps, world-level
  vehicles, rules/vars/HUD) and `quests.json`. No game-authored scripts — the
  build is native-player streamable.
- Characters (heron villagers, the wanderer, the catfish, the Mire Dragon, and
  the four camp enemy kinds) are Meshy-generated, rigged, web-optimized GLBs in `models/meshy/`,
  streamed at runtime (never packed into the `.pck`). Vehicles use Meshy
  multi-part bodies (rolling wheels, spinning prop) with parametric fallbacks.
- NPC dialogue is spoken and answered live by the shared NPC brain
  (`npc.myapping.com`).

## Build
```bash
godot --headless --path . --import
godot --headless --path . --export-release "Web" out/index.html
cp world.json quests.json out/ && cp -R models audio out/
```

## Credits
Environment/prop kits and audio: KayKit, Kenney, Quaternius, Fertile Soil
(CC0 asset library). Characters and vehicles generated with Meshy AI.
