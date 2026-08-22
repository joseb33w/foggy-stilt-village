# HERONWADE — Game-Feel / Mobile-UX Review

**VERDICT: FAIL (1 P0)** — the elder's hut cannot be entered, so quest step 2 (heron charm) and therefore the world goal `complete_quest: village_of_stilts` are unreachable.

Method: drove the actual web export headless (SwiftShader Chromium, `hasTouch` CDP touch events only — no keyboard for gameplay) at phone portrait **390×844** and landscape **844×390**, DPR-independence confirmed via the engine's short-side scale (viewport 720×1558 at both DPRs). ~10 full sessions, spawn → elder → hut door → pier → Old Murk, with `window.gogiGetPlayer()` telemetry and `window.gogiSolids()` collider probes. ~45 screenshots reviewed (kept in `/tmp/feel/*.png`). The live preview URL serves an app-install interstitial to mobile UAs, so runs used the local export with `/godot-assets/*` proxied from the preview host (real props load; only hut-interior furniture GLBs fall back to placeholders in-sandbox — absolute same-origin URLs, fine in production).

---

## ❌ P0 — The elder's hut is physically unenterable → quest/goal cannot complete

**Symptom.** The hinged door opens (USE → prompt flips to "Close Door"), but the player cannot cross the threshold. In **4 independent sessions** the player froze at exactly **z = 39.0–39.1** (the door plane) while pushing north, including dead-centred on the doorway gap (x 36.4–36.8; gap is x 35.8–37.2), door verified open, jumping included. `collect: heron_charm` (chest at ~(33.5, 33.5) inside) is therefore impossible, so the quest — and the world goal — can never complete. The completion panel/reward path is dead.

**Evidence (collider probe, door OPEN).** `gogiSolids` around the hut (world.json c2_2 structure, world footprint x 32–41, z 32–39):
- Door leaf collider **leaves the gap when opened** (probe of the gap window returns no leaf while prompt = "Close Door") — the door mechanic itself works.
- Doorway is collider-free below ~1.8 m (only the jamb posts at x 34.5/37.2 and the flat threshold ramp).
- One full-footprint solid body spans **x 32–41, y 1.83–4.98, z 32–39** — an upper-shell/gable-cap body whose collision dips to **head height (~1.83 m) over the door plane**. Player capsule top ≈ 1.9 m → blocked at exactly the observed plane. (Alternative/additional suspect: the threshold-ramp/slab lip. Either way it is engine-side geometry, not the data.)

**Where to fix.** `build_structure.gd` (enterable-shell path): the cap/upper-shell collider must not extend down into the doorway (keep roof collision ≥ wall top, or cut the door face out of it), then verify with a literal walk-through (`gogiGetPlayer` z crossing 39 → 37) rather than a door-opens check — the door opening is what masked this.

Note: `interior.door_face:"n"` lands the door on the boardwalk side (+Z) as intended — the face choice is correct; the blocker is the collider above the gap.

## ❗ P1 — All five `regions` are offset (−8, −8) from their landmarks (corner vs centre cell math)

Engine cell centre is `gx*16+8` (spawn confirms: start_cell [0,2] → player at (8,40)); the region centres in world.json were authored with corner math. Measured in play:
- **"The Reed Road"** (0,32) r10 is 11.3 m from the spawn/gate (8,40): the entry toast fired at boot in ~half my sessions and **never fired at all** in the others when walking the intended eastward path (PLACES SEEN stuck at lower counts).
- **"The Elder's Hut"** (28.5,27.5) r6 barely grazes the real hut (32–41, 32–39): `seen_hut` (+1 explored) fired for my bot **standing in open mud ~8 m SW of the hut** — twice — and PLACES SEEN reached 5/5 in a session where the hut was never entered. Conversely, standing inside the hut would NOT trigger it.
- **"Old Murk's Pool"** (94,32) r8 vs the fish at (102,40): the subtitle+shake fires ~8–11 m before the fish is even visible in fog, only because the swim path grazes the circle's edge; standing at the fish is outside the region.

**Fix.** Add (+8, +8) to every region centre in world.json; then re-centre "The Elder's Hut" on the actual hut (≈ (36.5, 35.5), r ~6, still covering the interior) and "Old Murk's Pool" on the fish (≈ (102, 40)).

## ❗ P1 — The only door's approach is congested: elder in the USE cone, wanderers body-crowding the frame, scatter on the path

- **Elder Sedge is authored 3.6 m in front of the door** ((36.5, 42.6) vs door (36.5, 39)): tapping USE on the door approach targets "Talk to Elder Sedge" instead of the door (nearest-wins, both within 2.9 m). Reproduced: opened chat when trying to open the door.
- The village-square **wander crowd loiters in/at the doorway** — repeated frames show 2–3 herons stacked on the player at the threshold, filling most of the frame with feathers at the game's key interaction point (the one place the camera reads "walled" in the whole game).
- A **scatter bush sits on the boardwalk against the doorway** and a **dead tree grows through the plank steps** in front of the door — visual blockers on the critical path (scatter wasn't audited against the door zone).

**Fix (data).** Move the elder ~3 m aside (porch corner), keep the populate wander radius/centre clear of the door face, and pull the bush/tree scatter off the walkway + door approach.

## ❗ P1 — Quest objective label is flaky: absent for entire sessions

In 3 of 6 full boots the top-left "QUEST: Village of Stilts …" label **never appeared** — the whole village walk played with no visible objective (the only guidance is one 5-second toast at start; spawn also faces away from the village, see polish). In the other 3 boots it showed from spawn and behaved well. Per the workspace engine source the label is chain-gated (`_quest_lbl.visible` only set when the mode declares a `chain`) and the wander mode declares none, so its appearance at all looks like a boot-order race in the shipped build.
**Fix (data-first).** Give the wander mode a `chain` for `village_of_stilts` so the label is deterministic; also consider showing only the active step — the current 4-step block wraps into ~8 cramped lines on 390 px and the region toast prints across it when both are active (seen at Old Murk and at the hut).

## ❗ P1 — Skerrin (and pier wanderers) stand in the channel water

Skerrin is authored at (92.8, 40.5); the canyon channel starts ≈ x 91 and the pier planks end ~x 86–88 — the "fisher on the pier head" stands shin-deep offshore (screenshot: heron standing in open water east of the pier), and pier-populate wanderers stroll into the channel (r6 around (88,40) reaches x 94). USE at the pier head produced no visible talk state. **Fix:** move Skerrin onto the pier platform (~(87, 40)) and tighten the pier populate radius.

## ⚠️ Polish

- **Spawn faces empty fog.** Default cam_yaw looks north into bare reeds; the boardwalk, gate and village (the entire game) are east, behind the camera, while the toast says "follow the boardwalks". Face the start camera east — the east-facing view (captured) is a genuinely inviting first frame.
- **Title screen:** "HERONWADE" wraps to "HERON / WADE" at both aspects (reads as two words); HUD bleeds through/over the title backdrop — "PLACES SEEN 0", the red HP bar sliver + stats remnant (the hide rule only runs at mode start) and ghost JUMP/USE are visible on the title, most noticeably in landscape.
- **Drawn weapon in a peaceful game:** the wanderer carries the `start_weapon` mallet/rusty-sword in hand through a combat-free village (combat HUD correctly hidden). Start sheathed or ship no weapon.
- **Snag pockets off the walkway:** concave prop pockets (e.g. crate+barrel+hut at ~(55, 46)) can momentarily wedge movement; a human recovers by backing out, but fat-finger joystick play brushes these often. Consider nudging dressing 1 m off the walkway edge.
- Mood drift: world.json ships `sky.time:"day"` (white-grey fog) vs the planned sunset amber; readability is fine, ambience is flatter than the brief.

## ✅ Passes (verified, not assumed)

- **Touch controls (the P0 class): all work one-handed at both aspects.** Virtual joystick moved the player (15.3 m / 9.9 m holds), right-side drag turned the camera (−1.1 rad per 100 px drag — brisk but in range), JUMP fired via touch (vy 3.0, anim "jump"), USE via touch talked to the elder, opened/closed the door, and addressed Old Murk. Button grid sits in right-thumb reach, entirely clear of the joystick half.
- **Camera:** no clip-through of huts/NPCs/the giant catfish in any reviewed frame; melee-range NPCs and the colossal Old Murk never fill the frame like a popup (the fish close-up is the best shot in the game — player visibly small against its flank, HUD and horizon intact; pushing INTO the fish still didn't wall the camera). Pitch clamps (−1.05 … 0.25 rad) never stare at floor/sky irrecoverably; both extremes are user-driven and recoverable.
- **HUD on phone:** portrait + landscape layouts clean — minimap square top-right, PLACES SEEN dodges it (engine `HUD_FIT` relocation observed), JUMP/USE never clipped (landscape short-viewport fix works), hidden-button rule respected (attack/weapon/potion/stable/cycle/stats/health absent everywhere), **no debug text in any frame** (GOGI telemetry is console-only).
- **Transient vs persistent:** region names ("The Reed Road", "The Village Square", "The Elder's Hut", "Old Murk's Pool") are 2.2 s engine toasts — confirmed faded in +8–75 s follow-up frames; start toast and charm toast fade; Old Murk's stir is a non-modal subtitle + 0.25 shake that cleans up fully (verified frame with zero stale text). Persistent set is exactly: quest label (when it shows), PLACES SEEN, minimap, JUMP/USE.
- **Pace/scale:** 6 m/s walk ⇒ gate→pier ≈ 13 s real time — right for a 128 m village; swim to Old Murk works (4.2 m/s), prompt reachable from the water.
- VRAM telemetry ~52 MB — comfortably inside the mobile budget (QA's gate, but no feel risk).

## Could not verify (sandbox limits)

- **NPC LLM chat replies/close flow:** chat POSTs are cross-origin-blocked in the sandbox; "Elder Sedge is speaking…" / "Old Murk is speaking…" states verified, replies not. Verify one reply renders non-modally on the live host.
- **Quest completion panel** ("THE FOG FEELS LIKE HOME"): unreachable behind the P0.
- True multi-touch feel (simultaneous move+look chords), real-device notch/safe-area (engine inset code present; insets were 0 headless), audio output, real-GPU frame pacing.

*Screenshot evidence for every claim above is in `/tmp/feel/` (portrait `p4-*`,`p5-*`,`p6-*`,`p8-*`,`p9-*`,`p12-*`; landscape `land2-*`; collider dumps `solids-*.json`).*
