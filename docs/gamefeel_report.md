# Game-Feel / Mobile-UX Report — Heronwade `feat/multiplayer` (CONTINUE: multiplayer + auth gate + respawn rule)

**VERDICT: FAIL (0 P0, 3 P1)**

Scope per delegation: the NEW surfaces only (boot/auth flow, sign-in form, post-sign-in sanity, respawn toast) at **390×844 portrait**. Verified by driving the real export (`/workspace/repo/out`, served locally) headless with **touch input** (Playwright `touchscreen` + CDP touch drags), Supabase proxied through Node per the known sandbox gotcha. Real sign-in with the verify account **succeeded through the actual touch-driven form** (`GOGI_AUTH gate passed uid=3f6dbd7c…`). Evidence frames in `/tmp/feel/*.png` (A2, B0–B4, D2–D7 referenced below).

---

## ❗ P1 — Sign-in form GHOSTS through the title screen (reads as broken z-fighting)

**Symptom (verified, screenshot A2):** ~12 s into boot, while the HERONWADE/WANDER title is still up, the sign-in form is faintly but clearly visible bleeding through it — the email/password field boxes, the drawn eye icon (brightest artifact, white @ 0.5 alpha), and the "Sign in" button text sit ghosted between the tagline and the WANDER button. On the very first screen every new player sees, this reads as a rendering bug.

**Root cause (code-traced):** `main.gd:461 _gate_auth()` runs mid-boot and `auth_gate.gd gate()` sets `visible = true` the moment the world build reaches it — independent of the title. The gate is on `layer 90`, the shell's title on `layer 150`, and the title backdrop is authored **semi-transparent**: `world.json → director.title.bg = [0.04, 0.06, 0.05, 0.94]` — 6 % of the form leaks through.

**Fix direction (pick one):**
- *Data-side (one value, game-owned):* set the title bg alpha to `1.0` in `world.json` (`director.title.bg[3]`). Kills the bleed outright.
- *Engine-side (cleaner ordering):* in `auth_gate.gd`/`main.gd`, don't set the gate visible while the shell's title is showing (main already exposes the director; `_title_root.visible` is the check) — reveal it on title dismissal.

## ❗ P1 — Death shows a full-screen DEFEATED modal ON TOP of the toast+respawn the new rule authored

**Symptom (code-traced; could not trigger live — see sandbox limits):** the new `mp_respawn` rule (`world.json` rules[9]: `player_died → toast "The marsh takes you... and gives you back." + respawn`) makes `rules.references_death()` return true (`rules.gd:134` — *any* `player_died` listener counts). The death path `main.gd:1279-1284` (duplicated at `~1414` in `set_player_health`) then does BOTH:
1. `fire("player_died")` → rule runs synchronously: toast + `respawn_player(true)` — player teleported to spawn, healed, `clear_defeat()` a no-op because the panel isn't up yet;
2. `_show_defeat()` → full-screen "DEFEATED / TRY AGAIN" modal (`game_shell.gd:640`, input-blocking, 0.85 dim on layer 150) appears OVER the already-alive, respawned player, dimming/hiding the toast (toast lives on hud_layer 0). "TRY AGAIN" calls `main.restart_run()` — a **full run reset** for a player who was already given back.

This is exactly the "modal popup on routine death" defect class, and it directly contradicts the authored copy ("…and gives you back"). It IS reachable in real play: PvP hits route through `take_damage` (`main.gd:3183-3195 _on_hit_taken`) and multiplayer is now enabled with 6 players.

**Fix direction:** make death non-modal, don't add a death screen — after `fire("player_died")`, only call `_show_defeat()` if the player is still dead (`rpg.hp <= 0.0`), i.e. no rule claimed the death by respawning/healing. Alternative: `references_death()` should only count rules that run `lose`, not any `player_died` listener. (Removing the rule is NOT a fix — it would lose the toast and revert to silent heal-in-place.)

**The toast itself is right:** transient label (fades via `_toast_t`, `game_shell.gd`), non-modal, eye-level at 16 % height, 80 % width; copy is in-voice and short. ✅ once the modal stops stomping it.

## ❗ P1 — Auth form touch targets render at ~24 pt, half the platform minimum the code itself targets

**Symptom (measured):** the project stretches a 720-wide base viewport onto the 390 pt screen (`canvas_items` + `expand`, scale = 390/720 ≈ 0.54; confirmed live — `GOGI_HUD_GRID vp 720x1558 win 390x844`). So the gate's "44 pt" fields (`auth_gate.gd:123` — comment says *"44pt: the smallest reliably tappable target"*) actually render **≈ 24 pt tall** (measured ~24 px field height in B0 at 390 CSS px width; same ratio holds on a real DPR-3 phone). Eye toggle ≈ 26×24 pt; "No account? Create one" / "Forgot password" flat buttons ≈ 17 pt tall, packed at ~6.5 pt gaps — adjacent-target mis-taps (Sign in vs Create one; email vs password) are likely on a thumb. This is a HARD gate every new player must pass one-handed; Apple/Android minimums are 44 pt/48 dp.

**Root cause:** `auth_gate.gd` sizes controls in Godot units assuming 1 unit = 1 pt; under this project's 720-base stretch every unit is worth 0.54 pt. **Fix direction:** scale the gate's metrics by the content-scale (or ~2× the constants: fields ≥ 82 units, eye ≥ 88×82, real buttons instead of flat text links), or put the gate on its own unscaled sizing.

---

## ⚠️ Polish

- **Region toast overlaps the quest log while showing** (D2): "The Reed Road" toast (y = 16 %) renders across quest line 4 top-left for its ~5 s life. Transient, so not a P1 — but nudging the toast below the quest block (or right-aligning it) would stop the momentary text-on-text. `game_shell.gd:843-845`.
- **"HERONWADE" wraps mid-word to "HERON / WADE"** on the title at 390 pt (84-unit font in a ~390 pt column, `AUTOWRAP_WORD_SMART` breaking inside the single word). Stacked it almost reads as a style choice, but it's an artifact; a smaller `name` font or authored line break would make it deliberate. Pre-existing surface, surfaced here because the boot flow is under review.

## ✅ Passes (all verified live via touch at 390×844)

- **Boot ordering coherent:** overlay tap → title → single WANDER tap → clean sign-in form (B0: no stray error, no double-title; "commit to play, then identity" reads fine for a first-timer, and the gate remembers the session afterwards per `Auth.restore_session`).
- **Form layout:** centered, no overflow, nothing clipped; fields at y≈378–426 sit comfortably in the top half — above where a virtual keyboard lands. "No account? Create one" and "Forgot password" both visible without scrolling.
- **Error state legible** (B1): "Enter a valid email address." in salmon, centered under the fields; clear-button (×) appears in the email field. Signup flip works and clears stale status (B3: "Create account" / "Have an account? Sign in").
- **Password reveal toggle works via touch** (B2: plaintext shown, drawn eye changes state — no missing-emoji box).
- **Real sign-in end-to-end via the touch-driven form:** gate passed, `GOGI_MP enabled max_players=6`, rules layer alive post-gate (`start_amb`, `seen_gate` fired).
- **Game unchanged post-sign-in:** touch joystick drag moves the player (frame + minimap shift, D2→D3); right-side touch drag yaws the camera (D4); JUMP and USE fire from their bottom-right thumb column via tap (D5/D6, buttons ≈ 119×65 pt — generous); pitch clamp holds on a full-screen downward drag — no floor-stare (D7); quest log readable against the fog (shadowed yellow); **no debug text on screen** (VRAM/HUD telemetry is console-only); no pinned region label — "The Reed Road" fades as a toast.

## Could not verify (sandbox limits)

- **Live death → toast/modal sequence:** no single-player damage source exists (no enemies; only a 1-damage weapon) and the PvP path needs a second peer — the container cannot open `wss://` (known env artifact; `GOGI_MP_NET peers=0` all run). The P1 above is code-traced with exact lines, not observed on screen.
- **Real multiplayer feel** (peer avatars, hit-direction arc) — same wss block.
- **Real-device keyboard occlusion, notch/safe-area, true multi-touch** (joystick + look simultaneously) — headless container has no virtual keyboard, notch, or concurrent touch streams.
- **Returning-player silent path** (`restore_session` skipping the form) — fresh browser context each probe run.
- Landscape aspect not judged — delegation specified 390×844 portrait.
