# HERONWADE — a foggy village on stilts

A peaceful exploration game for the phone browser. Wander a small fishing village
built on stilts over a swamp, follow the boardwalks through the fog, talk to the
tall heron folk who live there, step inside the elder's hut — and pay your
respects to Old Murk, the enormous ancient catfish asleep in the pool under the
great pier.

**Play:** https://preview.myapping.com/cloud-do09d2jdlpivbwyjcag8/index.html

## How to play
- **Move:** left-side touch joystick (or WASD)
- **Look:** drag the right side of the screen (or mouse)
- **Talk / open / interact:** USE button (walk up to a heron first)
- **Jump:** JUMP button (or Space)

The quest log follows *Village of Stilts*: meet Elder Sedge by the great hut,
take the heron charm from inside it, walk the great pier, and whisper a greeting
to Old Murk below the boards. Five named places toast as you discover them.

## Tech
- **Godot 4.7.1**, Compatibility (WebGL2) renderer, single-threaded (`nothreads`)
  web export — runs in Safari/Chrome/Firefox on mobile and desktop.
- Data-driven world on the `godot-tmpl-rpg` engine: everything that defines the
  place lives in `world.json` (chunk-streamed 8×8 grid, terrain + swamp water +
  fog weather, parametric stilt huts, one enterable interior) and `quests.json`.
  No game-authored scripts — the build is native-player streamable.
- Characters (three heron villagers, the wanderer, the giant catfish) are
  Meshy-generated, rigged, web-optimized GLBs in `models/meshy/`, streamed at
  runtime (never packed into the `.pck`).
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
(CC0 asset library). Characters generated with Meshy AI.
