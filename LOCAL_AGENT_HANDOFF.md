# Local Agent Handoff — Phaser Angry Birds Clone

**File:** `/root/game/index.html` (single-file Phaser 3 + Matter.js game)  
**Last fixed by:** Grok 4.3 (xAI) on 2026-05-30  
**Status:** Playable after critical physics fix. Local models can now iterate safely.

---

## What Was Fixed

The game was broken because of incorrect Matter.js API usage:

- **Broken calls** (would do nothing):
  - `this.matter.world.setVelocity(...)`
  - `this.matter.world.setAngularVelocity(...)`
  - `this.matter.world.applyForce(...)`

- **Correct calls** (now used):
  - `this.matter.body.setVelocity(body, velocity)`
  - `this.matter.body.setAngularVelocity(body, angularVelocity)`
  - `this.matter.body.applyForce(body, position, force)`

These were the root cause of birds not launching properly, power-ups having no effect, and explosions not pushing objects.

**All instances were updated** using targeted search/replace. The rest of the codebase (collision, damage, levels, SFX, input, win conditions) was already in good shape.

---

## Current Game State (Summary)

- 7 levels defined in `LEVELS` array
- Birds, pigs, blocks, TNT, ground
- 4 power-ups (bomb, ice, wind, speed)
- Slingshot launching with elastic bands
- Collision-based damage + destruction
- Custom oscillator SFX (no external assets)
- Matter physics with gravity, bounds, collisions
- Camera follows active bird
- Win condition: clear all pigs

---

## Recommended Next Version Goals (Prioritized)

1. **Polish & Stability First** (do these before new features)
   - Tune launch power, gravity, and restitution for better "feel"
   - Improve pig/block destruction thresholds and scoring
   - Make sure every power-up works reliably on both birds and blocks
   - Fix any remaining edge cases in `onCollision` or `update`

2. **Quality of Life / Engagement**
   - Add a simple score + stars (1-3) per level
   - Add level select screen or progression
   - Improve mobile touch handling (current input is functional but can be refined)
   - Add more visual feedback (better particles, screen shake on big explosions)

3. **Content Expansion** (only after the above is solid)
   - Add 3–5 new levels with increasing difficulty
   - Introduce 1–2 new bird types or power-ups
   - Add a "retry" or "next level" flow after winning/losing

**Do NOT** rewrite large sections of the file unless necessary. Prefer small, targeted edits.

---

## Guardrailed Prompt for Local Agent (Copy-Paste This)

Use the following prompt when giving tasks to the local model:

---

**SYSTEM PROMPT FOR LOCAL AGENT (copy everything below this line)**

You are working on a single-file Phaser 3 + Matter.js Angry Birds clone located at `/root/game/index.html`.

**Strict Rules (never break these):**

1. **Matter.js API** — You must ONLY use these correct calls:
   - `this.matter.body.setVelocity(body, {x, y})`
   - `this.matter.body.setAngularVelocity(body, value)`
   - `this.matter.body.applyForce(body, position, force)`
   Never touch `this.matter.world.setVelocity`, `setAngularVelocity`, or `applyForce` again.

2. **Single-file discipline** — Keep everything in `index.html`. Do not create new files unless explicitly told.

3. **Incremental changes only** — Make one small, testable change at a time. After each edit, verify the change by reading the modified section.

4. **Preserve existing patterns** — Use the same style as existing code:
   - Constants are defined near the top
   - Level data lives in the `LEVELS` array
   - SFX calls go through the `SFX` object
   - Collision logic lives in `onCollision`

5. **When stuck** — If you cannot make a feature work after 2–3 attempts, stop and write a clear note at the bottom of this handoff file explaining exactly what you tried and what failed. Do not keep hacking.

6. **Testing mindset** — After any physics, collision, or input change, describe what should happen and confirm the code change matches that intent.

**Current task template:**

"Improve [specific thing, e.g. 'bird launch feel' or 'add score system']. Make the smallest possible change that achieves the goal. Follow all the strict rules above. After editing, show me the exact lines you changed and explain why."

**End of system prompt for local agent.**

---

## How to Use This File Going Forward

1. Give the local agent the guardrailed prompt above + a specific task.
2. Let it work until it either succeeds or gets stuck.
3. When it starts producing broken or repetitive output ("barfing"), paste the current state + error here and bring it back to Grok.

This structure should let the local model handle the majority of iteration safely.

---

**Last updated:** 2026-05-30 by Grok 4.3  
**Next escalation trigger:** Local agent produces non-functional code or violates the Matter.js rule.

---

## 2026-05-30 Local Qwen 3.6-27B Update — Next-Level Feature Pass

**Files changed:** `/root/game/index.html`

**Added features:**
- Added differentiated bird types via `BIRD_TYPES`:
  - `red` / Classic: balanced default bird.
  - `heavy` / Crusher: larger, denser, slower launch, 1.75x collision damage.
  - `speedy` / Comet: smaller, faster launch, lighter impact.
- Added per-level `birdTypes` loadouts so each level now has a designed bird queue instead of identical birds.
- Updated HUD bird icons to show bird type color/labels (`H` for Crusher, `S` for Comet).
- Updated bird drawing so type visuals combine with existing power-up visuals.
- Updated launch physics to use each bird type's density, restitution, and launch multiplier while preserving the correct `this.matter.body.*` Matter API.
- Updated collision damage so bird type damage multipliers affect blocks/TNT/pigs.
- Added floating score text for block destruction (`+100`) and pig kills (`+1000`).

**Verification performed:**
- Extracted inline JavaScript and ran `node --check /tmp/game_inline.js` — passed.
- Served `/root/game` locally with `python3 -m http.server 8088 --directory /root/game` — HTTP 200 OK.
- Static feature/API checks passed:
  - Phaser CDN present.
  - `BIRD_TYPES` present.
  - Per-level `birdTypes` present.
  - Floating score text present.
  - Correct `this.matter.body.setVelocity/applyForce` calls retained.
  - Forbidden `this.matter.world.setVelocity/applyForce/setAngularVelocity` calls absent.
  - Complete HTML present.

**Verification limitation:** the Camofox browser tool was unavailable (`Cannot connect to Camofox at localhost:9377`), so no interactive browser launch test was possible in this session. Next agent should manually play level 1 and verify: HUD queue shows red/red/H, heavy bird appears as a larger gray bird, impacts feel stronger, and score popups appear when blocks/pigs are destroyed.

**Suggested next pass:** add active bird abilities (tap-to-slam for Crusher, tap-to-dash for Comet) or add limited power-up inventory counts per level.