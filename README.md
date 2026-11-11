# Choices — Post-Apocalyptic Survival

A single-file 3D post-apocalyptic survival game starring Kes, set 72 hours after the grid went down. Drive a car around an open village, climb ladders, survive fires and falls, make quiz-like decisions in thought bubbles, deal with animals and an angry mob.

## Play it

Open `index.html` in any modern browser — no build step, no server needed. Or deploy the repo to Vercel / GitHub Pages.

## Source

Built in Claude and exported 2026-11-11. `index.html` (14.4 MB) is the complete published game — a standalone React build in one file.

Note: the chat's last message says work paused when Claude free usage ran out, while Claude was adding a character named Michelle chasing with procedural animation — so this build is the latest published version and does **not** include that update. **Decision (2026-11-11, Kester): Michelle is dropped entirely — forget her. No named characters. NPCs/chasers are random male and female characters, no names needed.**

## The gameplay bar (from Kester's Claude chats)

This is the standard every iteration is judged against:

- **Driving must feel like Far Cry 2:** see the front of the car, drive from the driver's seat with a real interior, proper engine-acceleration sound, brake screech, no reversed steering after a crash, no glitchy/blinking view while driving. Top speed around 11.5–13.
- **Visuals must be realistic, not boxy or flat:** every boxy element (animals, cars, trees, props) replaced with rounded, Blender-modelled versions with real photo textures (brick, rust, bark, feathers). Upgrade the world as a whole, not piecemeal.
- **Animals must move and sound real:** limbs actually move and gaits match speed; animals make their correct real recorded sounds (not synthesized noises), including when bumped into, and stay visible.
- **Combat/mechanics:** machete swing like Far Cry 2; an angry mob of people carrying bats appears from a corner or at random while exploring; ladder climbing like Call of Duty (walk up facing it, hold W, look around, E/Space to let go); fire drains health (16/sec); falls cause damage (none under 3 m, ~half health from a short building, lethal from ~12 m).
- **Structure/feel:** stage 1 stays in daylight — save nighttime for a whole different stage (only stage 3 "NIGHT 3 — BROKEN BODY" is night); start screen text big, bold and near-white with light orange-tinted frosted glass behind the middle text only; decision pop-ups use the same readable font and orange glass.
- **Process:** preview a new model (e.g. the chicken) BEFORE it is committed to the game, and changes must actually show up in the preview.

## Character direction (2026-11-11, Kester's verdict on the coded preview)

- **Coded preview built:** `preview/character-preview.html` — unnamed survivor + wounded variant (walk/run/limp, bandage, blood), built entirely from smooth code shapes, self-contained. Live: https://kes10011.github.io/choices/preview/character-preview.html
- **Kester's verdict: "we could work with that style i guess… even though it looks like what we got already."** The coded character is acceptable as a fallback, but it is the same visual tier as the current game characters — NOT the realism jump he wants. **Decision (2026-11-11, Kester): "we'll go with your style" — the coded style is the chosen direction for the characters.** Realistic external models remain an option for animals (see `asset-hunt.md` — dog: FeralMotion Labrador).

Iterations continue in Muse (Zyphar). Edit `index.html` directly — it is the whole game. Commit and push; Vercel auto-deploys if the repo is connected.
