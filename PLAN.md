# Goal
Keep the village's existing foggy daytime sky as the dominant look, and add a looping
thunderstorm cycle: fog → clouds build → thunderstorm (rain, lightning, thunder) → rain
tapers → back to fog.

# Files to touch
- `world.json` — replace the fixed `"sky": {"time":"day","weather":"fog"}` with a looping
  `cycle` whose first segment is the same day/fog condition (load-in look unchanged).
- `main.gd` — engine re-sync from the current rpg template (template-owned, no game logic).

# Verification approach
- Headless Godot export (nothreads web), verify.mjs smoke: engine boots, console clean, frames captured.
- Targeted check: long-run headless browser session sampling a frame inside the storm segment
  (~t+95s) to confirm the storm actually arrives (darkened sky, rain particles).

# Out of scope
- Gameplay, quests, models, multiplayer, audio assets — unchanged.
- No backend changes (data-only weather cycle).
