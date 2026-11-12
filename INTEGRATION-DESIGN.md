# Choices — Asset Integration Design Doc

**Status:** DRAFT v1 (Elephant) — 2026-11-11 ~15:50 AST; revised 2026-11-11 ~20:00 AST
(Goldfish 3/3 resolved) and ~20:20 AST (Goldfish 1/3 + 2/3 resolved); revised 2026-11-12
~00:20 AST (Grab3D Woman §2a(i) + Grab3D extras §2a(ii) + road/clutter design + honest
~37 MB projection). **ALL THREE GOLDFISH REVIEWS RESOLVED — the doc is now vetted and buildable.**
**Author:** coordinator (Elephant session). **Review:** Goldfish 1/3, 2/3, 3/3 all resolved 2026-11-11.
**Goal:** get every CGTrader download Kester picked into the game, replace the boxy/primitive
models with realistic ones, make buildings enterable, add the door-knob glow, and improve
progression + flow — without breaking anything that already works. **Scope (Kester,
2026-11-11): the upgrade covers the WHOLE game including Stage 1 — that is why all of the
assets were downloaded. Characters, animals, trees, grass, buildings, vehicles, weapons —
everything upgrades everywhere it appears, Stage 1 first among them. The chase stage is
one part of the upgrade, not the whole of it.**

**Standing rules (non-negotiable, from Kester):**
- Nothing boxy. Rounded models + real photo textures. Upgrade the world as a whole, not piecemeal.
- Realistic external models ONLY. Coded characters are OUT (rejected 2026-11-11). Never re-pitch AI avatars.
- NO named characters. NPCs/chasers are random male and female characters, no names.
  Michelle is DROPPED entirely — never re-add her in any form.
- Preview each new model BEFORE it is committed to the game.
- The gameplay bar in `README.md` stays intact unless this doc explicitly improves it.
- Free tools/models only. Never pay.
- **No car belongs to anyone.** The world is in shambles — any car anyone can drive.
  Vehicles are FOUND, not owned — the villains drive what they find too. Never write
  "Kes's car" / "his car" / ownership language for any vehicle (Kester, 2026-11-11). The
  Jeep Renegade is the car Kes FINDS and drives in the chase; the Generic Old Car 04 is a
  roadside wreck because it is a shitty car nobody would choose to drive, not because it
  belonged to anyone.
- Commit + push per completed swap (targeted adds only, never `git add -A`).
  Every push goes live at https://kes10011.github.io/choices/ automatically.
- Nothing is "done" until verified on the live link Kester sees.

**Standing design principle — supplies make you a target (Kester, 2026-11-11):**
"The game is apocalyptic survival choices… in real life you'll be attacked for your stuff."
Carrying more makes the player a bigger target: every time the player picks up supplies,
that is a choice between having more and being more of a target. This runs wider than the
chase stage — it governs the whole game: supply pickups (§6 item 1) feed the player's stock,
and the world's hostility (mob spawn pressure) scales with how much the player is carrying.
The chase stage (Appendix A) is the sharpest expression of it: the road pirates chase Kes
BECAUSE he has stuff worth taking — his supplies (food, water, fuel) in the car — and the
stake is keeping the stuff, not just health: win = reach the village gate WITH the supplies;
lose the car / get caught = lose the stuff.

**Standing combat rule — fight or flight on foot (Kester, 2026-11-11):**
On foot, when attacked — whether by people (mob, NPCs) or by animals (dog and any other
hostile animal) — the player has two options: (1) **defend himself and kill the attacker**,
or (2) **run away far enough for the attacker to RESET** (disengage and return to where it
came from). Fleeing is always a valid option and costs nothing — but the threat stays there
for next time. Fighting costs health/ammo but removes the threat permanently. The reset
distance must be READABLE to the player: the attacker visibly gives up and heads back.
This is the fight-or-flight rule for ALL on-foot encounters. (It does not apply inside a
vehicle — the chase stage is the in-vehicle exception: no go-on-foot alternative until the
checkpoint, Appendix A.) See §6 item 13 for the implementation.

**Out of scope for this build (do NOT build):**
- The CHASE STAGE (villains on bikes/vehicles trying to run Kes off the road) is PROPOSED
  ONLY — see Appendix A for the full draft concept. The concept is essentially SETTLED
  (Appendix A §A5) but still NOT approved for build — no chase-stage code until Kester
  approves it. (The 4 new vehicles make it possible; the stage itself is not part of this
  build.) The settled chase places constraints on the vehicle choices: the truck must be
  promotable to a war-rig, the bike must work as a pirate vehicle, the Jeep is the chase car.
- The archviz scene (514 MB Blender scene) — convert-or-drop per §4 (extract exterior
  meshes if clean, else DROP). The medieval block (Lumion `.ls11`) — DROPPED, no usable
  format (Blender cannot import it; no Lumion available).
- The audiobook / any FinFit work — unrelated, ignore.

---

## 1. How the game works today (verified by reading the bundle)

`index.html` (14,417,545 bytes) is one minified React + Three.js bundle. No build step:
the builder edits `index.html` directly.

**Structure — 4 stages** (`wD`):
| Stage | Title | Mission | Decisions |
|---|---|---|---|
| 1 | DAY 1 — DRY MOUTH | Find safe water and food | water "The Well" (7, 48) · food "The Pantry" (-14.8, 38) |
| 2 | DAY 2 — COLD NIGHTS | Get warm without getting seen, and judge the stranger | fire "Alley Fire Site" (-21, -44) · stranger "The Stranger at the Gate" (2.2, 7.2) |
| 3 | NIGHT 3 — BROKEN BODY | Moonlit night: treat the injury and find proper shelter | meds "The Field Lab" (15.8, -44) · shelter "Where to Sleep" (17.8, -38.5) |
| 4 | DAY 4 — THE WAY OUT | Work the radio and signal the rescue | radio "Emergency Radio" (-14.6, -68) · signal "The Helicopter" (6, -56) |

Each decision = 3 choices (rank 0/1/2) with effects `{health, thirst, hunger, warmth, battery}`
+ hours cost + outcome text + lesson text. Decisions trigger by walking to a floating marker
and interacting (E). When both stage decisions are done → "Mission complete" toast, a green
flare appears at the north checkpoint (0, -102) → walk there to advance. `advanceStage()`:
next day 06:00, hunger AND thirst floored at 45, warmth floored at 60, heals +15, respawns
the player at (0, 1.7, 90), `mobs.clearAll()`, `spawnAmmo()`. Win = reach the checkpoint on
Day 4. Night is stage 3 only (`isNight` = stage === 2). **Player spawns at (0, 1.7, 264)** —
the code has `const dD=180` and the spawn is `(0, 1.7, 84+dD)`; world clamp x ±112,
z -112..294 (114+dD). **The map is ~224 m wide × ~406 m long, NOT ~224 m square.** All
decision-marker coordinates in the table above are in this shifted frame (they match the
`AA()` call args) — state the frame explicitly when placing anything new.

**Stats:** health / thirst / hunger / warmth / battery. Death causes: dehydration, starvation,
freezing, fire, mob, fall. Fire = **16/sec** within 1.05 m (matches the bar). Fall:
`fallDamage(A)`: A ≥ 12 m = death; else health -= (A-2.5)*9 (a 5–6 m fall = 22–31 dmg —
softer than the bar's "~half health from a short building"; see §6 item 7).

**Animals** (`class tS`): cow ×12, dog ×10, cat ×6, chicken ×18, birds ×9 (primitive-built).
Feral: dog 40%, cow 30%. HP: cow 90 / dog 40 / cat 20 / chicken 12. Animals wander, flee
when bumped, feral ones attack. Sounds: 5 recorded MP3s embedded (cat, cow, dog, hen,
rooster). Engine + fire sounds are SYNTHESIZED (noise buffer + bandpass filter) — NOT
recorded. Kester's bar wants a proper engine-acceleration sound: known gap, see §6 item 14.

**Mobs** (`class lS` + `LS()` builder): spawn 3–6 per group from building corners 42–95 m
away, first spawn at 28 s, then `spawnT = max(35, 85 − stage*10) + random*55` — and the
timer ONLY ticks while zero mobs are alive (no new group spawns while any mob lives).
`advanceStage()` calls `mobs.clearAll()` with `spawnT=40`. HP 55, states walk/chase/windup/
recover, weapon random (cleaver/bat/pipe/crowbar, dmg 7–9). **Mobs are built from
PRIMITIVES** — box torso/limbs, sphere head + cap. This is the single biggest "boxy"
violation left in the game, and the main thing the human models fix. **The mob update loop
writes six animation slots unguarded every frame** — `legL/legR` (gait), `armL`
(counter-swing), `armR` (windup −2.5 / recover), `torso` (rotation.z/x), `head`
(rotation.x=.18 on chase). There is no optional chaining and no fallback: a mob GLB that
lacks ANY of the six slots throws a `TypeError` every frame or leaves the mob frozen. The
adapter MUST provide all six (see §3b). Hitting a mob before it swings costs **−10 health
(−25 if killed)** via `unprovoked()` — the morality system punishes unprovoked attacks
(see §6 item 4: the Day 4 surge group must spawn provoked, or the climax punishes the
player for fighting it).

**The Wounded Stranger** is a primitive-built sitting man (`P6()` builder, torso node animated
for breathing) propped against the wrecked sedan at (2.2, 7.2). He is the emotional peak of
Day 2 — swapping him to a realistic seated figure is high priority.

**Viewmodels** (first-person hands/weapons, `this.vms`): light / machete / gun — ALL
primitive-built (`bS`/`OS` builders, flat colours). Kester: "my flashlight and machete and
gun suck." The new shotgun / machete / flashlight models go HERE (plus world pickups).
The gun viewmodel needs a **muzzle anchor node** (the muzzle-flash effect attaches to it).

**Safe house = POSITION ZONE, not geometry.** `inShelter()` checks the player is inside a
cottage's rectangle (w/2-.35, d/2-.35), on foot, below 3 m height. Cottages are solid boxes
with collision; "inside" is a zone check. Truly enterable buildings need real door openings,
interior wall colliders with a doorway gap, and the zone logic extended (see §4).

**Driving:** car enter/exit (E near car, <3.6 m), W/S drive, A/D steer, SPACE brake, E step
out. First-person view shows the car interior + steering wheel node (`getObjectByName(
'steering wheel')`). Weapons + flashlight hidden while driving. Ammo 18 start, 2 caches per
stage, weapons 1/2/3 = light/machete/gun. Map (M): WATER / FOOD / WARMTH / SHELTER / AMMO.
**THREE drivable cars already exist** — pickup (-6.5, 44), sedan (6.5, -62), station wagon
(6.5, 250). The Jeep is the FOURTH drivable, not the second. The station wagon sits ~15 m
from the Day-1 spawn (0, 264) — the most discoverable car in the game (see §6 item 6).
The driving code has ONE global clamp (`speed = clamp(speed, -4, 11.5)`), one acceleration
curve, and one steering formula — no per-car fields. Per-vehicle handling (top speed,
accel, steering) needs per-car parameters added to the car object (see §3b). The driving
camera depends on the car GLB's `'steering wheel'` node, reparented to `userData.wheel` as
the driver-seat anchor; if the car GLB fails to load, `drive()` builds a procedural cockpit
and the seat height changes (`hasCar ? 1.2 : 1.55`). **There is no chase camera anywhere
in the game** — it is first-person only (see §3b: the Jeep/Norton camera decisions).
`hitByCar()` kills any mob at speed >4 for 200 damage (mobs within 2.2 m of the car).

**World:** main road runs north–south (z axis) between two dead villages; cottages built
procedurally (`A6()`) with variants (burnt/moss/dirty) and a door side (`side`, e.g. "W") —
only cottages WITH a door side count as shelters. 4400 instanced grass tufts. Trees:
3 variants rebuilt from the trees GLB with leaf textures. Extraction flare + green point
light at (0, 0, -104). Start screen: "72 HOURS AFTER THE GRID WENT DOWN", "BEGIN — DAY 1".

---

## 2. Inventory — landed files (verified 2026-11-11)

Extracted to `~/workspace/choices/work/extract/`. Source of every file: CGTrader free models,
downloaded via the browser task with Kester's `kesterakawariyah@gmail.com` sign-in.
**Licenses: VERIFIED per model — the browser task's final report is the parent's inventory
below (every file verified on disk 2026-11-11 ~20:30 AST, author + license per file).**
CGTrader free models carry VARYING licenses — Royalty-Free is NOT the default. All landed
models are Royalty Free EXCEPT Maxtree (CUSTOM LICENSE — free for personal + commercial
use; record its exact license text in the swap log). The builder records source URL +
license per model in the README swap log (§11) and does NOT ship any model whose license
is unclear or non-commercial. **§9 item 0 is SATISFIED for the 23 landed files by this
table** — the builder may commit swaps for landed models. The DOG (not yet downloaded) is
the only pending license.

### 2a. Landed — FINAL inventory (23 files, ~2.6 GB, verified by the parent 2026-11-11
~20:30 AST; browser task DONE — hit CGTrader's daily free-download limit of 25, resets
tomorrow). 21 models (Norton = 2 files, Military Truck = 2 files). Session A:
`~/workspace/browser_downloads/sess-371879480/` (1.9 GB). Session B:
`~/workspace/browser_downloads/sess-1125121562/` (637 MB). All 23 files copied to
`~/workspace/choices/work/incoming/`.

| # | Archive (as landed) | Model | Contents (verified) | Size |
|---|---|---|---|---|
| 1 | `worker_caucasian_fbx.rar` (19,595,381 B) | **Worker Character** (male) | adam127 | Royalty Free | FBX | 34,990 tris | Rigged MakeHuman. No walk/run anims listed, no GLB. |
| 2 | `FBX.7z` (92,975,574 B) | **Wild Stylized Girl** (female) | GFMFS | Royalty Free | FBX | 70,497 polys | Rigged Auto Rig Pro + FaceIt. STYLIZED per her own title, not realistic. **SUPERSEDED 2026-11-11 ~20:06 AST (Kester): she is NOT the female mob — FALLBACK ONLY.** Rigged keeper exists (`preview/models/girl.glb`, 1,483,116 B, 16,515 verts, 1.69 m, all 6 slots verified) and goes back into the pool sparingly (~1 in 4) ONLY if the Grab3D Woman Base Mesh fails her preview-gate walk test (§2a(i)). |
| 3 | `Zane_Ryder_fbx.zip` (106,385,991 B) | **Zane Ryder** (extra mob) | mahgadx | Royalty Free | FBX | 32,771 polys | 4K textures, no GLB, rig/anim not listed. |
| 4 | `Hen.rar` (1,026,662 B) | **Hen** (free low-poly) | promiseijalade | Royalty Free | FBX | 1,062 polys | Rigged PBR. Labelled low-poly; the most realistic FREE hen on CGTrader (photorealistic hens are paid $15–$41). |
| 5 | `Gledista_Triacanthos_FBX.zip` (161,281,775 B) | **XfrogPlants Honey Locust** | xfrog | Royalty Free | FBX | 1,125,789 polys total (6 tree models) | Wind-animated. HEAVY — needs aggressive decimation. |
| 6 | `MT_PM_V060_FBX.7z` (727,220,583 B) | **Maxtree Plant Models Vol 60 grass** | Maxtree | CUSTOM LICENSE (free for personal + commercial use) — NOT Royalty Free | FBX | 15 species / 90 models | Record the exact license text in the swap log. Only the 6 grasses are used. |
| 7 | `glb.rar` (59,377 B) | **New York Buildings** | rickardosanchez | Royalty Free | GLB | 3 GLBs (~100–138 KB each) | **CORRECTED IDENTITY:** these ARE the New York Buildings — the 3 GLBs inside are named `L_build_1/2/3` (each: 3 nodes LOD0/1/2, 3 meshes, NO materials, NO animations). EXTERIOR-ONLY filler/skyline — description states "the models do not contain the internal parts of the buildings." |
| 8 | `export_concrete_house_003.zip` (90,509,033 B) | **Medieval House (Enterable Long Hut)** | Enterables | Royalty Free | OBJ/FBX/3DS (no GLB) | 75,596 polys, 3 LODs | **CORRECTED IDENTITY:** this is NOT a "Concrete House 003" — it is the enterable Medieval House (Long Hut). ENTERABLE, interior EMPTY/unfurnished. Use the FBX (better than OBJ for export). |
| 9 | `-t_hut_007_export.zip` (108,644,883 B) | **Medieval Wooden Silo Windmill (Enterable)** | Enterables | Royalty Free | OBJ/FBX/3DS (no GLB) | 10,868 polys, 3 LODs | **CORRECTED IDENTITY:** this is NOT a hut/cabin — it is the enterable Windmill. ENTERABLE, interior EMPTY/unfurnished. Use the FBX. |
| 10 | `Medieval_Block_9.ls11` (663,592,772 B) | **Lumion Scene Medieval Block** | syembs | Royalty Free | LUMION 11 SCENE FILE ONLY — NO FBX or GLB version | — | EXTERIOR-ONLY street scenery. **DECISION: DROP for this build** — `.ls11` is a proprietary Lumion format Blender cannot import; no Lumion available. Cannot be a backdrop either (no extractable geometry). Revisit only if Lumion access appears. |
| 11 | `small_village_fbx.zip` (65,635,175 B) | **Small Village with Two Fully Furnished Homes** | drezeldeen-tantawi | Royalty Free | FBX | Single 71.5 MB FBX | ENTERABLE — 4 homes: 2 fully furnished, 2 empty (1 closed, 1 broken door), plus trees/grass. Verify the count + interiors + separable furniture in Blender before §9 step 8. |

Session B — `~/workspace/browser_downloads/sess-1125121562/` (637 MB):

| 12 | `Proside.zip` (514,853,211 B) | **Blender Cycles Free Archviz Scene** | proside | Royalty Free | BLENDER SCENE ONLY — NO FBX/GLB version | 514 MB `.blend` | Exterior scene/filler, no interior shown. **DECISION: convert-or-drop** — the builder opens the `.blend` in Blender; if exterior building meshes extract cleanly at reasonable poly counts, use 1–2 as skyline/backdrop fillers; otherwise DROP. |
| 13 | `Remington_model_31_GLB.zip` (36,902,590 B) | **Shotgun Remington Model 31 in 3 Types** | blackmax | Royalty Free | GLB (already!) | 6,160 polys | Rigged + reload animation, 3 states (New/Worn/Old) — pick **Worn** (fits the apocalypse). No FBX→GLB conversion needed; still needs decimation/resize to the §7 budget. |
| 14 | `Machete.fbx` (71,536 B) | **Machete** | Kevingm2002 | Royalty Free | FBX | 989 polys | 4 handle colours. Its separate 479 MB texture pack was NOT downloaded — verify textures in Blender; if untextured, apply a flat material. |
| 15 | `SM_Flashlight_01.FBX` (111,680 B) | **Flashlight Yellow** | diolator | Royalty Free | FBX | 2,060 polys | PBR. |
| 16 | `Norton_Manx_Models.zip` (57,200,931 B) + `Norton_UV_Textures.zip` (557,631 B) | **1947 Norton Manx motorcycle** | jloubser | Royalty Free | BLEND/FBX/OBJ/3DS/DAE | 404,291 polys | HEAVY. Chain is separate pieces so it can be rigged. Prefer the BLEND (may have better rigging) or FBX. UV textures in the separate zip. |
| 17 | `Jeep_Renegade_2016_fbx.zip` (471,705 B) | **Jeep Renegade 2016** | Jose-Bronze | Royalty Free | FBX | 4,734 polys | Real photo textures. Light. **THIS is the car Kes finds and drives in the chase** (no car belongs to anyone — vehicles are found, not owned). |
| 18 | `FBX.rar` (14,800,603 B) | **Generic Old Car 04** | Mavi3D | Royalty Free | FBX | 295,000 polys | High-quality exterior + medium interior. HEAVY — needs decimation. One commenter reported textures did not appear in their download — verify on extraction; if missing, flat materials. Demoted to roadside wreck/scene prop only. |
| 19 | `gltf.glb` (16,200 B) + `blend.blend` (19,463,719 B) | **Military Truck (6x6 Armored Tactical)** | RavixStudio | Royalty Free | GLB (16 KB) + BLEND (full version) | — | The GLB is only 16 KB so the BLEND is the full version — use the BLEND as the source (open in Blender, export GLB); verify the 16 KB GLB in Blender first (may be too low-detail to use directly). Rigged wheels, animated. Candidate for the villain war-rig blocker. |
| 20 | `ShepherdValley_Cow_FBX.zip` (14,896,882 B) | **Low Poly Cow (Holstein, rigged, animated)** | RetroStyleGames | Royalty Free | FBX | 3,814 polys | **5 animations**, PBR 2K textures. Light. (The code drives legs procedurally; the 5 baked anims are a bonus — note them.) |
| 21 | `achara_vuelo_simple.fbx` (8,266,652 B) | **Achara Bird (small flying bird)** | pixol3d | Royalty Free | FBX | 22,905 polys | Rigged + animated in flight. Its separate 30 MB `tex.rar` texture pack was NOT downloaded — verify textures in Blender; if untextured, apply flat colours / the game's bird textures. Heavy for a bird — needs decimation. |

### 2a(i). Grab3D — Woman Base Mesh (landed 2026-11-12 ~00:06 AST; NOT CGTrader — no daily-limit slot)

**Woman Base Mesh** — https://grab3d.com/models/characters/woman-base-character/ — **CC0 1.0** (public domain, no attribution, no login; LICENSE.txt saved at `work/incoming-grab3d/woman-base-mesh/LICENSE.txt`). **SHE IS THE FEMALE MOB FROM THE START (Kester, 2026-11-11 ~20:06 AST): "get everything tonight from Grab3D so there is NO swap tomorrow — use the right girl from the start."** The earlier decision ("ship the cartoon Girl sparingly, swap tomorrow", 2026-11-11 ~20:00 AST) is SUPERSEDED — the cartoon Girl does not go into the pool at all unless this woman fails the gate.

| File | Size | Verified |
|---|---|---|
| `woman-base-character-rigged.glb` (keeper) | 12,902,608 B | Rigged: 1 skin ("Armature", 24 joints), animations **"walk" + "run" (72 channels each)**, 163,993 POSITION verts, ~300,000 tris, 3 images/textures, 1 material, 1.68 m tall A-pose, grounded at Y=0, dark brown hair in bun, taupe sports top + charcoal shorts, bare feet. Copies: `work/incoming/woman-base-character-rigged.glb` + `work/incoming-grab3d/woman-base-mesh/`. |
| `woman-base-character.glb` (static) | 13,261,824 B | 0 skins, 0 animations, 305,702 tris. Reference only — do NOT use in game. |
| `woman-base-character-light.glb` (NOT downloaded) | 4,864,288 B | Light variant (152,851 tris, 1024 px textures) listed on the page manifest — available if the full rig's decimation struggles; still needs decimation to ≤18k verts. |

**Conversion — PREVIEW GATE PASSED 2026-11-12 ~00:30 AST (coordinator-verified with my own eyes):** keeper `preview/models/woman.glb` — **1,140,196 B, 16,818 verts / 21,000 tris** (≤18k verts / ≤1.5 MB — PASS), sha256 `89b674e9…`. Pipeline: Blender `convert.py` REJECTED for this rig (Blender 4.5's glTF importer mangles her skeleton — bone heads thousands of cm off; `convert.py`'s route gate is identity and would NOT have caught it); used the Worker route (gltf-transform weld → simplify 0.07 → textures 512 JPEG → JSON-only bone renames → `bake_armature_scale.py`, IBM max error 1.07e-05). 6-slot rename: LeftUpLeg→`legL`, RightUpLeg→`legR`, LeftArm→`armL`, RightArm→`armR`, Spine→`torso`, Head→`head`. **Config row for the pool:** legs rest 3.10 (x≈π — mob loop MUST write rest-relative), arms axis `x`, armR sign +1 (same as the Girl: idle −1.15 → hand fwd/up, windup −2.5 → overhead 1.70 m), NO z-lower constant (A-pose already), headY 1.43, socket offset (hand in armR-local) (0, 0.46, 0.07). Walk test passed (frame sequence `work/qc/woman-walk-side/` — flat planted front foot, toe-off, bent knees, counter-swinging arms; no paddling, no floating). **Walk gate (§8):** PASSED — she IS the female mob. The pool swap (replace the `mobGirlGLB` const + this config row) is instructed to the step-4 builder; in-game verification waits on the invisible-mob fix. **If she fails in-game**, the coordinator tells the step-4 builder explicitly and the cartoon Girl goes back in sparingly as the fallback — the builder does NOT put the Girl in on its own initiative. **AI disclosure:** the mesh is AI-assisted (Meshy image-to-3D, cleaned in Blender — see LICENSE.txt); counts as AI-assisted if a platform requires disclosure (e.g. Steam). The game ships on GitHub Pages, so no disclosure is currently required — record it here so it is not lost.

### 2a(ii). Grab3D extras — trees, road, ground clutter (landed 2026-11-12 ~00:10 AST; all CC0 1.0, all grounded Y=0)

Fetcher done (agent e6c238f0), verified on disk by the parent: 11 GLBs, ~57 MB, in `work/incoming-grab3d/` (full table: `work/incoming-grab3d/INVENTORY.md`; each model has its own LICENSE + manifest + NOTE alongside). **All CC0 1.0, no login, no CGTrader slots touched.** Pine/road/bushes/rocks/dead-trees are DONE — no CGTrader slots needed for them.

| Role | File | URL | Size | Verified | Verdict |
|---|---|---|---|---|---|
| Pine / conifer tree | `pine-tree/pine-tree.glb` | https://grab3d.com/models/nature/pine-tree/ | 949,600 B | 8,032 tris, 5.7 m tall, grounded | **KEEPER → pine slot `$6`/`n.scene` (§3a)** |
| Dead tree | `dead-trees/dead-tree.glb` | https://grab3d.com/models/nature/dead-tree/ | 1,639,960 B | 13,849 tris, 4.5 m tall, grounded | **KEEPER → dead-tree slot `AS`/`b.scene` (§3a)** |
| Tree stump | `dead-trees/tree-stump.glb` | https://grab3d.com/models/nature/tree-stump/ | 690,496 B | 8,264 tris, grounded | **KEEPER → ground clutter (§3b)** |
| Road segment (asphalt) | `road-segment/road-segment-straight.glb` | https://grab3d.com/models/city/road-segment-straight/ | 5,537,920 B | 4,220 tris, 11.8 m tile | **KEEPER → main road (§3b). NOTE: 5.5 MB = heavy textures — compress to ≤512 px before embedding.** |
| Dirt road | `road-segment/dirt-road-light.glb` | https://grab3d.com/models/city/dirt-road/ | 1,589,284 B | 24,551 tris (light version), 11.8 m tile, rural fit | **KEEPER → village paths + chase back-roads (§3b)** |
| Bush | `bushes-rocks/bush.glb` | https://grab3d.com/models/nature/bush/ | 1,410,856 B | 11,570 tris, ~1.1 m, grounded | **KEEPER → ground clutter (§3b)** |
| Boulders | `bushes-rocks/boulder-rock-set.glb` | https://grab3d.com/models/nature/boulder-rock-set/ | 751,484 B | 10,112 tris, grounded | **KEEPER → ground clutter (§3b)** |
| Scatter rocks | `bushes-rocks/small-rock-set.glb` | https://grab3d.com/models/nature/small-rock-set/ | 730,564 B | 6,656 tris, grounded | **KEEPER → ground clutter (§3b)** |
| Dog (candidate) | `dog-candidate/dog.glb` | https://grab3d.com/models/animals/dog/ | 15,998,932 B | **STATIC — 0 skins, 0 anims, 329,190 tris** | **REJECTED as a swap — not rigged.** Kept on disk as reference only. German Shepherd (CGTrader) remains the pick, first slot tomorrow. |

Skipped by the fetcher, with reason: spruce-tree (heavier than pine), snowy-pine-tree (snow), berry-bush (berries wrong for a wasteland), rocky-outcrop (heavier single piece; the sets cover it), spooky-tree (stylized), wolf (not a dog, 800k tris). Do not re-pick them.

### 2b. Still expected + per-swap dependency rule (Goldfish fix — replaces the old blanket ban)
**Rule:** there is NO blanket "do not start until everything lands". Each swap depends on
specific files; a swap whose file hasn't landed is SKIPPED with a clearly-marked README
TODO, and all other swaps proceed in §9 order. The build starts when Blender is verified
(§9 step 1), not when the last download lands. The per-swap table below is re-checked at
build start and the "Landed?" column updated.
**Browser task DONE** (hit CGTrader's daily free-download limit of 25, resets tomorrow).
**Still expected — ONLY the DOG** (German Shepherd, RetroStyleGames, first priority
tomorrow): https://www.cgtrader.com/free-3d-models/animal/mammal/german-shepherd-3d-model-6eb1ecb6-0808-4398-92d8-51b1951c60a1 — Royalty Free, FBX zip
`RSG_DogsPack_GermanShepherd_FBX.zip`, 18.5 MB, rigged, 8 animations (walk, run with
leans, idle), 2K PBR, 2,272 polys. One review notes the mesh around the neck is broken.
**Not yet selected or downloaded:** nothing — pine/conifer tree, road segment, bushes/rocks/dead
trees all landed from Grab3D tonight (§2a(ii), CC0, no CGTrader slots needed). **Still open
for tomorrow's CGTrader run: German Shepherd dog ONLY (first slot).**
**REJECTED — do not re-pick:** Walking People Pack (free, Royalty Free, but 5 static
unrigged posed models, no animations — not hero material); Amber female model (Editorial
license, "Not for commercial use" — rejected); stylized 3D Chicken Model and faceless
Ginger Female (rejected).

| Swap (§9 step) | Required file | Landed? (2026-11-11 ~20:30 AST) |
|---|---|---|
| Hen → chicken (step 2) | Hen.rar | ✅ YES |
| Old Car 04 → wreck (step 3) | FBX.rar (Generic Old Car 04) | ✅ YES (295k polys — decimate; textures may be missing) |
| Mobs + Stranger (steps 4–5) | Worker / Grab3D Woman / Zane | ✅ YES (Woman landed 2026-11-12 ~00:06 AST — §2a(i); cartoon Girl is fallback only) |
| Weapons → viewmodels (step 6) | Shotgun (GLB) / machete / flashlight | ✅ YES (shotgun already GLB) |
| Trees + grass (step 7) | Honey Locust / Maxtree grasses | ✅ YES |
| Enterable buildings (step 8) | Medieval House ✅ · Windmill ✅ · village ✅ | ✅ YES (all three landed — identities corrected, §2a rows 8/9/11) |
| Vehicles (step 9) | Jeep / Norton / Military Truck / Old Car 04 | ✅ YES |
| Fillers (step 10) | NY buildings ✅ (glb.rar = L_build GLBs) · Medieval Block ❌ DROPPED (Lumion, no usable format) · Archviz — convert-or-drop | PARTIAL (block dropped, archviz TBD) |
| Pine → pine (step 7) | pine-tree.glb (Grab3D, CC0) | ✅ YES (8,032 tris, 5.7 m — §2a(ii)) |
| Dead tree → dead tree (step 7) | dead-tree.glb (Grab3D, CC0) | ✅ YES (13,849 tris, 4.5 m — §2a(ii)) |
| Road + ground clutter (step 7) | road-segment-straight / dirt-road-light / bush / boulder-rock-set / small-rock-set / tree-stump (all Grab3D, CC0) | ✅ YES (§2a(ii)) |
| Dog → dog (step 12) | German Shepherd FBX (CGTrader) | ❌ NO — tomorrow, first slot. Grab3D dog is STATIC (0 skins, 0 anims, 329k tris) — REJECTED as a swap, kept on disk as reference only. |
| Cow / bird (slot into steps 2/7) | Cow / Achara bird | ✅ YES |
| Dog (step 2/7) | German Shepherd | ❌ NO — tomorrow, first priority. The current dog (Cube) stays until then. |

### 2c. Vehicles — page facts + decisions
| Vehicle | Page facts (checked on CGTrader) | Decision |
|---|---|---|
| **1947 Norton Manx** (motorcycle) | Blender 2.7, 119 objects, **404,291 faces**, UV textures, OBJ/FBX/DAE/3DS. Chain has individual pieces (rigging possible — wheels can spin). Minor clipping on piping/wiring. | **RIDEABLE — Kes rides it.** Same driving bar as the car (Far Cry 2 feel, proper handling, no reversed steering). Decimate to ≤50k faces. Arcade handling matched to the car, lean visual on turns, no tip-over death. Rider pose: seated, hands on bars. |
| **2016 Jeep Renegade SUV** | FBX, **4,734 polys**, real photo textures, low-poly, "very light and fast to render". Author: José Bronze. | **THE CAR KES FINDS AND DRIVES IN THE CHASE — primary drivable + chase vehicle (Kester, 2026-11-11: "nah swap that shitty car lol").** The Jeep replaces the sedan as the chase car. Kes finds it after a stage mission (chase gate) and drives it in the chase. No car belongs to anyone — it is found, not owned. Handling: heavier, top speed ~10, more stable. OPEN: interior for first-person driving view NOT confirmed on the page — verify on download; if no interior, chase cam for the Jeep only (the sedan interior is part of the sedan GLB and cannot be reused). |
| **Generic Old Car 04** | FBX, **295,000 polys**, high-quality exterior + **medium detailed interior**, real dimensions, textures included, FBX/OBJ, objects separated and named. Author: Mavi3D. **One commenter reported textures did not appear in their download — verify on extraction; if missing, flat materials.** | **WRECK / SCENE PROP ONLY — demoted (Kester, 2026-11-11).** A broken-down roadside car fits the apocalypse. It is NOT drivable and NOT the chase car — it is a shitty car nobody would choose to drive (not because it belonged to anyone: no car belongs to anyone). Replaces the current wreck (`W6`/`c.scene`) and appears as roadside wrecks. The Wounded Stranger sits against a wrecked sedan — this is the natural model for that scene. Decimate hard (295k polys → ≤20k). |
| **Military Truck** | **6x6 Armored Tactical**, RavixStudio, Royalty Free. **GLB (16 KB) + BLEND (19.5 MB, full version)** — rigged wheels, animated. | **SCENE / WRECK VEHICLE + VILLAIN WAR-RIG CANDIDATE.** Military convoy wreck as a landmark + cover + mission site (search the truck bed for supplies — §6 item 1 bandage site). Promotable to the villain war-rig blocker in the chase stage (Appendix A). Use the BLEND as the source; verify the 16 KB GLB first (may be too low-detail). |

---

## 3. Swap map — which new model replaces which embedded GLB

The 18 embedded GLBs are base64 consts (`u6,J6,p6,d6,F6,Z6,iS,TS,q6,h6,H6,W6,f6,X6,_6,$6,AS,PS`),
parsed at load and destructured to `[P,v,g,Q,j,i,T,s,t,M,B,c,U,E,r,n,b,y]`. The swap replaces the
base64 string of a const with the new model's base64 (or adds a new const + loader entry for
brand-new objects). **The builder verifies each replacement by decoding the old GLB and the new
GLB and comparing node names, bounding boxes and scale BEFORE touching `index.html`.**

**Slots are identified by CONST NAME + destructured name ONLY — never by bracket number.**
(The v1 draft's bracket numbers mixed two different enumerations and pointed at the wrong
consts for chicken / cow / wreck / oak / shed / fire / cottage / barrel / crate / pine /
dead / far. The table below is ground truth: every GLB decoded from the bundle 2026-11-11
and cross-checked against the code's own model object — `{car:P.scene, dog:cg(Q.scene,.95,…),
cat:cg(j.scene,.55), bird:cg(i.scene,.7), shed:cg(T.scene,7), cottage:B.scene, oak:r.scene,
pine:n.scene, dead:b.scene, far:y.scene, wreck:c.scene, barrel:U.scene, crate:E.scene,
chicken:cg(t.scene,.48), cow:cg(M.scene,2.6), fire:cg(s.scene,1.8)}`.)
**Surprise the builder must know: the "dog" slot (`d6`/`Q.scene`) is a single Cube mesh — the
game builds the dog from a cube.** The dog swap replaces that cube with a real dog GLB in the
same slot; nothing else about the dog code path changes.

| Const | Var | Role in code | GLB contents (decoded from the bundle) | Size |
|---|---|---|---|---|
| `u6` | `P` | car (sedan) | body, seats, dashboard, exhaust | 2,492,040 B |
| `J6` | `v` | trees | 3 variants (leaves/tree ×3) | 2,152,652 B |
| `p6` | `g` | house | door, wall corners, roofsupports, planks | 1,445,352 B |
| `d6` | `Q` | **dog** | single `Cube` mesh — the dog IS a cube | 119,508 B |
| `F6` | `j` | cat | quadruped (Head, Neck, paws, arms) | 94,060 B |
| `Z6` | `i` | bird | bird (`KeyAction` ×3) | 190,168 B |
| `iS` | `T` | shed | single node `001` | 864,480 B |
| `TS` | `s` | fire (campfire) | Campfire + fire_part_00–28 (fire anims) | 904,408 B |
| `q6` | `t` | chicken | body, beak, comb; material `feather` | 157,804 B |
| `h6` | `M` | cow | torso, patch; materials `hide`/`patch` | 366,476 B |
| `H6` | `B` | cottage | wall/floor structure (wallL/R/B, floor, lintel) | 115,072 B |
| `W6` | `c` | wreck (car #2) | lower, hood, trunk, cabin, roof | 89,280 B |
| `f6` | `U` | barrel | barrel, rings, lid | 20,784 B |
| `X6` | `E` | crate | post/rail structure, used as the crate | 49,344 B |
| `_6` | `r` | oak | bark trunks (`deadbark`) | 44,528 B |
| `$6` | `n` | pine | bark + cones (`farleaf`) | 9,080 B |
| `AS` | `b` | dead (tree) | bark + leaves (leafA/B/C) | 34,676 B |
| `PS` | `y` | far (tree) | bark + leaves (pineA/B) | 16,620 B |

### 3a. Direct replacements (same slot, same role)

| Slot (const → var) | Old model | New model | Compatibility / adapter work |
|---|---|---|---|
| `q6` → `t.scene` | chicken (154 KB) | **Hen** | **ADAPTER REQUIRED — BONE RENAMES, NOT EMPTY PROXIES.** Game code needs material `"feather"` and nodes `legL`, `legR`, `head`, and it ROTATES those nodes to animate the legs (`gait()`). **An Empty parented to a bone is inert on a skinned mesh** — vertices follow bone matrices only, so rotating an Empty rotates nothing in the mesh. The fix is to rename the BONES themselves in Blender: `Bip01_L_Thigh`→`legL`, `Bip01_R_Thigh`→`legR`, `Bip01_Head`→`head`. Bone renaming is safe for skinning (skinning is index-based, not name-based). Then verify in the preview that rotating each renamed bone swings on the expected axis — a bone's local X axis is generally NOT the primitive group's swing axis. Material: rename `mati_chicken` to `feather` (keep the roughness/metallic maps + flat diffuse color [191,127,0] — there is NO diffuse texture). Which Hen meshes carry `mati_chicken` (beak/feet/comb may be separate materials) is listed from the Blender inspection — only meshes whose material is named `feather` get the colour tint in `swap()`. If the `.blend` route is used, verify its textures are packed first (it may reference unpacked external textures). Scale: `cg(t.scene, .48)` scales the FOOTPRINT (max(x,z)=.48), not the height — only `cg(..., P=true)` scales by height. Measure the Hen's bounding box and set the scale so the bird's footprint is ~0.5 m across. If the FBX has a baked walk/peck take, keep it named and note it; the game drives legs procedurally, so baked clips are a bonus, not a requirement. Sound: kind stays `"chicken"` → hen/rooster MP3 already wired. |
| `h6` → `M.scene` | cow (357 KB) | **Cow (LANDED — ShepherdValley Holstein, RetroStyleGames, 3,814 polys, 5 animations, PBR 2K textures)** | Same bone-rename pattern as the hen: rename the leg bones to `legFL/legFR/legBL/legBR` and the head bone to `head` (**NO Empty proxies — they are inert on skinned meshes**). Materials `"hide"`/`"patch"` are material renames (fine). **5 baked animations are a bonus, not the plan** — the code drives legs procedurally; note them in the swap log. If the cow model has no leg bones to rename, the swap is blocked until a rigged cow is found or the code path changes. Scale: `cg(M.scene, 2.6)` is a footprint, not a height. |
| `d6` → `Q.scene` | dog (116 KB, **single Cube mesh** — the dog IS a cube) | **Dog (NOT YET DOWNLOADED — German Shepherd, RetroStyleGames, first priority tomorrow; 2,272 polys, 8 animations, 2K PBR). The current dog (Cube) stays until then — the dog swap is deferred to a follow-up commit after the download lands. Grab3D's dog was checked and REJECTED (STATIC — 0 skins, 0 anims, 329,190 tris; kept on disk as reference only, `work/incoming-grab3d/dog-candidate/dog.glb`).** | The old "dog" is one Cube; the swap replaces it with a real dog GLB in the same slot. Dog/cat use the shader rig (`rig()` — bounding-box leg swing, NO node names needed). **Candidate dog GLBs must pass the shader-shear check in §8** — the shader swings vertices below 55% height, and a skinned/rigged GLB dog may double-animate or shear under it; "any quadruped-shaped dog GLB works" is overconfident. Verify the shape reads as a dog at 0.95 footprint (`cg(Q.scene, .95)`) and the leg-swing shader doesn't shear the head. Fallback if it fails: static pose (no leg swing) — do not ship a sheared dog. One review notes the mesh around the neck is broken — check the neck in Blender before the swap. |
| `Z6` → `i.scene` | bird (185 KB, `KeyAction` ×3 anims) | **Achara Bird (LANDED — pixol3d, 22,905 polys, rigged + animated in flight; its 30 MB `tex.rar` texture pack was NOT downloaded — verify textures in Blender; if untextured, flat colours / the game's bird textures)** | Game plays `birdClips[0]` if present, else static. If the new bird has a baked flap clip, keep it as clip 0. If it has no animation, the builder adds a code wing-flap (rotate wing nodes) — do NOT ship a frozen bird. (The old "relies on the primitive birds' wing planes" fallback is wrong: the current bird GLB has `KeyAction` anims, not wing planes.) Decimate (22,905 polys is heavy for a bird — §7 budget ≤300 KB GLB). |
| `W6` → `c.scene` | wreck / car #2 (87 KB) | **Generic Old Car 04 (LANDED — Mavi3D, 295,000 polys; decimate hard to ≤20k; one commenter reported textures did not appear in their download — verify on extraction; if missing, flat materials)** | Direct swap. Verify scale vs the current wreck (measure bounding box) and orientation (front faces the same way — the stranger sits against its side at (2.2, 7.2)). Interior exists (medium detail) — good for the stranger scene. |
| `_6` → `r.scene` | oak tree (43 KB) | **Honey Locust** (1 of the 6 variants) | Pick the best-looking variant by eye in the preview. **The `G` tree array CANNOT be extended to a 4th variant this way** — `G` is built by cloning nodes named `leaves`/`tree` with suffixes `''`, `.001`, `.002` from the TREES GLB; the Honey Locust is a separate GLB with different node names and no `.003` variant. A second Locust variant needs its OWN loader path (own node names, measured-height scaling) — write it separately and leave the `G` suffix loop untouched. The oak uses `r.scene` RAW (no `cg()`), so the Locust must match the oak's native scale, not a `cg()`-normalized one. **One embedded const, referenced twice — the oak replacement and the 4th tree variant share the SAME base64 const; do NOT duplicate the bytes.** |
| `$6` → `n.scene` | pine (bark + cones) | **Grab3D Pine (LANDED — CC0, 8,032 tris, 5.7 m, grounded Y=0; §2a(ii))** | Direct swap, same pattern as the oak: the pine is used RAW (no `cg()` — verify how the current pine is scaled in the bundle before touching it; if it uses `cg()`, match the footprint call). Verify the pine reads as a conifer at game distance (5.7 m source height vs the old pine's in-game height — measure, don't assume). One const, instanced/cloned wherever pines stand. |
| `AS` → `b.scene` | dead tree | **Grab3D Dead Tree (LANDED — CC0, 13,849 tris, 4.5 m, grounded Y=0; §2a(ii))** | Direct swap. Same scale check as the pine (measure the old dead tree's in-game height first). The dead tree is the wasteland's signature silhouette — verify it reads bare/dead (no leaves) in the preview at distance. One const, instanced/cloned wherever dead trees stand. |

### 3b. Additions (new consts / new code paths — no old slot removed)

| New object | Source | Integration |
|---|---|---|
| **Mob humanoids** (replaces `LS()` primitive builder) | Worker Character (male) · **Grab3D Woman Base Mesh (female — §2a(i), CC0, rigged, walk+run baked)** · Zane Ryder (extra) | New loader entries + a mob-model pool. Mob groups get a MIX of the three bodies (random male/female, no names). **Adapter — BONE RENAMES, one scheme, decided now.** The mob update loop writes SIX slots unguarded every frame (`legL/legR` gait, `armL` counter-swing, `armR` windup −2.5 / recover, `torso` rotation.z/x, `head` rotation.x=.18 on chase) — no optional chaining, no fallback. A mob GLB missing ANY slot throws a `TypeError` every frame or leaves the mob frozen. **The doc chooses scheme (i): the Blender adapter maps real bones to the exact slots the loop expects.** Rename the bones in Blender per body: thigh groups → `legL`/`legR`, upper-arm groups → `armL`/`armR`, `spine_02` → `torso`, head bone → `head`. (Bone renaming is safe — skinning is index-based.) NO Empty proxies for animation — an Empty parented to a bone is inert on a skinned mesh. **Empties ARE still fine as static attach points** (they follow the bone they are parented to): the weapon socket is an Empty parented to the `armR`/hand bone — the primitive builder parents the weapon to an arm child, and the new hands need the same attach point. **Headshot heights:** `ray()` uses hardcoded heights 1.64/1.1 m, not the head node — re-derive them from the model (or from `M.h.head` world position) per body, or headshots misalign on taller/shorter mobs. **Animation:** the builder drives the walk procedurally (rotate the renamed bones with `sin(phase)` in the mob update loop, mirroring the existing `gait()` pattern; swing arms opposite; lean `torso` forward slightly on chase). **Baked walk clips are NOT the plan** — the code has NO AnimationMixer for mobs (mixers exist only for birds/fire), so "prefer a baked walk clip" is unimplementable without adding a mixer (a scoped code change the builder does NOT make unless the coordinator approves it). The Woman HAS baked walk + run clips (§2a(i)) — they are the bake source if the procedural walk looks stiff (§10 item 1), not a runtime clip. Verify in Blender before deciding, and verify in the preview that rotating each renamed bone swings on the expected axis. **Walk gate (§8/§9):** the mob GLB must expose all six animation slots + the weapon socket, verified by GLB JSON + a headless walk-cycle frame-sequence capture — the coordinator judges it; stiff = reject, bake a walk cycle in Blender (§10 item 1). Mob HP (55), speed, weapon (cleaver/bat/pipe/crowbar — keep the existing weapon meshes for now; the new models' hands hold them via the weapon socket) and spawn logic are UNCHANGED. **Mob weapons stay primitive in this build** (no CGTrader mob-weapon models were ordered) — see the residual-boxy list in §3c. |
| **Wounded Stranger** (replaces `P6()` primitive sitter) | Worker Character, seated pose | High priority — this is the Day 2 emotional peak. Pose: seated, back against the sedan, one leg bent with the rag/blood detail. Implement the pose in Blender on a SEPARATE COPY of the Worker file: rotate spine/hips/thighs, then Pose → Apply Pose as Rest Pose (glTF exports the rest pose + animations — a Pose-Mode pose does NOT survive export otherwise, and applying it as the rest pose destroys the rig for reuse, hence the copy). Keep the breathing animation — drive it through the renamed `torso` bone — BUT retune the baseline: the current `oP.rotation.x=.25` was tuned for the primitive torso and will permanently hunch the seated figure on a `spine_02` bone. Retune in the preview. |
| **Shotgun viewmodel + world pickup** | Remington Model 31 (LANDED — GLB already, blackmax, 6,160 polys, 3 states; pick **Worn**) | Replaces the primitive gun viewmodel (`vms.gun`). **Needs a muzzle anchor node** (the muzzle-flash effect attaches to it) — add an Empty named `muzzle` at the barrel tip in Blender (an Empty is fine here: it is a static anchor for the flash effect, not a skinned bone). **Viewmodel integration spec:** viewmodels are built SYNCHRONOUSLY by primitive builders (`bS`/`OS`) returning `{g, muzzle}`; the gun's rest position/rotation are hardcoded in `vmBase` (gun at (.22, −.2, −.32)) and the muzzle-flash class consumes `i.muzzle`. The builder PRELOADS the shotgun GLB at startup and the viewmodel builder CLONES the loaded scene (it cannot parse a GLB synchronously). Per-weapon scale + `vmBase` retuning verified by first-person screenshots. **World pickup: DELETED as specified** — the player already owns all three weapons (keys 1/2/3, ammo 18 at start), so a pickup granting a weapon they already have does nothing, and there is no pickup code path for it (`nearestSupply` handles only POI and ammo; caches are re-randomized every stage by `spawnAmmo()`, so there is no fixed "near" to put anything). If a shotgun pickup is wanted, it must grant something real (e.g. +6 shells) with a defined `nearestSupply` kind, radius, and fixed coordinates — decided in §6 item 1, not here. Keep the existing gun stats (ammo, damage, shotCd) — this is a visual swap, not a balance change. |
| **Machete viewmodel + world pickup** | Machete (LANDED — Kevingm2002, 989 polys; 479 MB texture pack NOT downloaded — verify textures in Blender) | Replaces the primitive machete viewmodel (`vms.machete`). Same loader pattern as the shotgun (preload at startup, clone into the synchronous builder; `vmBase` retuning per weapon). The swing animation is code-driven (viewmodel rotation) — the new model must be oriented so the blade points the same way as the old one (verify in the preview against the old viewmodel's rest pose). World pickup: same as the shotgun — deleted as specified (the player already owns the machete). |
| **Flashlight viewmodel** | Yellow flashlight (LANDED — diolator, 2,060 polys, PBR) | Replaces the primitive light viewmodel (`vms.light`). Same loader pattern as the shotgun (preload, clone, `vmBase` retuning). The actual light cone (`this.flashlight` spotlight) is unchanged — only the in-hand model swaps. Verify the model has a lens/front face the light can plausibly come from; if the model is yellow plastic, keep the yellow — it reads on camera. |
| **Norton Manx** (rideable) | 1947 Norton Manx (LANDED — jloubser, 404,291 polys, BLEND/FBX/OBJ/3DS/DAE, UV textures in separate zip) | NEW vehicle system. Reuse the car driving code path with a bike variant: same W/S/A/D/SPACE/E controls, same no-reversed-steering rule, arcade handling, top speed ~12 (between car 11.5–13). **Camera decision (settled): first-person, handlebars + front wheel visible, NO visible rider body** — the same rule as the car (interior + wheel, no body). A visible seated rider and a first-person view cannot both be true; the first-person view wins. **Lean = camera roll in first-person** (the car code already rolls the camera: `rotation.z = -s*R*.004`) — specify lean as a camera-roll value, not a model roll. First-person anchors (handlebar node name, camera height/offset) are specified at build time from the model — the bike has no steering-wheel node, no cockpit, no seat height, so the builder defines them. **Per-vehicle parameters (new — the driving code has one global clamp, no per-car fields):** top speed, accel, steering factor, seat height + anchor node, camera mode — added to the car object per vehicle. Wheels: the chain/wheels have individual pieces — verify the front/rear wheel nodes are separable so they can spin with speed. **Decimate 404k faces → ≤50k.** **Spike-test:** the Norton is one of the riskiest conversions (119 objects, 404k faces) — spike-test it FIRST (§9 step 1), before the Hen proves the pipeline. **Settled-chase constraint:** the bike must work as a pirate vehicle (Appendix A). |
| **Jeep Renegade** (the car Kes finds and drives) | 2016 Jeep Renegade (LANDED — Jose-Bronze, 4,734 polys, real photo textures) | NEW entry in `world.cars` (the car system already supports a list — `nearCar()` iterates it). **THE CAR KES FINDS AND DRIVES IN THE CHASE — primary drivable + chase vehicle (Kester, 2026-11-11: "nah swap that shitty car lol").** Kes finds it after a stage mission (chase gate) and drives it in the chase. No car belongs to anyone — it is found, not owned. **Per-vehicle parameters (new):** top speed ~10, accel, steering factor (heavier, more stable = less slide), seat height + anchor node, camera mode — added to the car object (the driving code has one global clamp, no per-car fields). **Camera decision (settled):** first-person IF the Jeep has an interior (verify on download). **If it has NO interior, the Jeep is NOT drivable in this build as a first-person vehicle — the fallback is a scoped chase-camera build** (third-person camera behind + above the vehicle, same W/S/A/D controls), explicitly scoped as a new camera system, NOT a silent "decide at build time". The builder does NOT silently ship a chaseless Jeep, and does NOT "reuse the sedan interior framing" (the sedan interior is part of the sedan GLB and cannot be reused). **Interior gate:** the Jeep's drivability is gated on the download inspection — interior confirmed → first-person chase car; no interior → chase-cam scope decision, with Kester told plainly. Add a map POI for it once discovered (see §6 item 6). The three existing cars (pickup, sedan, station wagon) stay drivable in free roam. **Settled-chase constraint:** the Jeep must work as the chase car (Appendix A). |
| **Military Truck** (scene/wreck) | Military Truck (LANDED — RavixStudio, BLEND full version + 16 KB GLB; use the BLEND as the source) | Place as a roadblock at (0, -80), angled across the main road, with collision — see §4 placements. It blocks the road for VEHICLES (full-width; cars cannot pass) with a ≥1.5 m pedestrian gap on the west side, so the last ~20 m to the checkpoint is always on foot. Mission site: searching the truck bed gives a bandage pickup — reach-from-ground radius 2 m at the tailgate (the bed is ~1.5 m up; no climb mechanic exists; the pickup sits at the tailgate, outside the truck's collision footprint — `nearestSupply` measures 2D distance). NOT drivable in this build. **Settled-chase constraint:** the truck must be promotable to a villain war-rig (Appendix A) — keep its GLB intact for that. |
| **Medieval house** (enterable) | Medieval House (Enterable Long Hut) (LANDED — Enterables, 75,596 polys, 3 LODs, empty interior) | See §4. |
| **Windmill** (enterable) | Windmill (Enterable Wooden Silo) (LANDED — Enterables, 10,868 polys, 3 LODs, empty interior) | See §4. |
| **Small village** (3 enterable homes) | Small village (LANDED — drezeldeen-tantawi, 4 homes: 2 furnished, 2 empty (1 closed, 1 broken door)) | See §4. |
| **New York buildings** (filler — = the L_build GLBs) | New York buildings (LANDED — glb.rar, rickardosanchez, Royalty Free) | Skyline/backdrop at the map edges (beyond the playable clamp, x ±112 / z -112..114). No collision needed beyond the map bounds. No interiors. **The 3 GLBs are named `L_build_1/2/3` inside the files — the old "L_build" row and this row are the SAME models.** They have NO materials — the game's retexturing (`Rw()`) keys off material names, so materials must be AUTHORED in Blender (assign brick/plaster materials with names the game can key off), not "applied from a set". Check in Blender for vertex colours first. Use LOD0 in-game (LOD1/2 are for distance — the game doesn't do LOD switching, so ship LOD0 only). |
| **Concrete House 003 — DELETED as a separate row** | `export_concrete_house_003.zip` IS the enterable Medieval House (Long Hut) — see the Medieval house row above. There is no separate modern-house filler. | — |
| **Maxtree grasses** (terrain) | 6 grass FBXs | Replace the instanced grass-tuft geometry: the game draws 4400 instanced tufts from ONE `InstancedMesh` with ONE geometry (0.9×0.7 plane) and ONE material (alphaTest .35, DoubleSide). **One InstancedMesh cannot mix species "randomly per instance".** Each Maxtree species is a multi-mesh, multi-material plant — it must be MERGED/BAKED into one geometry + one material PER SPECIES (merge step in §7), then one InstancedMesh per species (2 species: Cynodon + Zoysia — the smallest, most grass-like), per-plant budget ≤2k polys, instance count ~1500–2500 total (full 3D plants cost more than crossed planes; ~2,000 × ~2k polys ≈ 4M polys needs the §9 fps gate). Material: alphaTest .35 handling per species (Maxtree grasses are typically alpha-tested leaf cards). Keep the same placement rules (the `gA`/`iA` exclusion zones around buildings/roads). |
| **Honey Locust** (tree variety) | 1–2 of 6 variants | See the oak row in §3a: one Locust variant replaces the oak const (`_6`/`r.scene`); a second variant needs its OWN loader path (the `G` suffix loop cannot be extended). Do not embed more than 2 variants. |
| **Road tiles** (NEW — no existing road geometry) | Grab3D road-segment-straight (asphalt, 4,220 tris, 11.8 m tile) + dirt-road-light (24,551 tris, 11.8 m tile) — both CC0, §2a(ii) | **One embedded const per tile type; tiles are placed as clones/instances along the road — NEVER one const per tile.** Asphalt tiles pave the main drivable road (the highway route, including the chase fork's highway branch, Appendix A); dirt tiles pave village paths and the chase back-roads fork. **Builder maps the existing road path FIRST** (from the terrain visuals + the three car positions + the truck roadblock at (0, −80) + the checkpoint) and lays tiles to cover it — the game has no road-path data to read, so the path comes from a top-down screenshot + the known landmarks, verified by a top-down screenshot BEFORE the road commit. Straight tiles only — no curves in the set; bends are faked by yawing successive tiles in small steps. Tiles are ground dressing: NO collision change (the road is already drivable terrain), NO gameplay change. Texture work: the straight road is 5.5 MB of heavy textures — compress to ≤512 px (§7 budget ≤500 KB GLB per type). **MEASURED 2026-11-12 ~00:35 AST (coordinator-verified, gate PASSED):** both tiles are **11.8 × 11.8 m SQUARES**, not strips — the straight tile is a full street section (asphalt + white edge/centre markings + granite curbs + concrete-paving sidewalks on both sides); the dirt tile is twin ruts through grass. Tile base Y=0, overall top +0.35 m (straight) / +0.42 m (dirt). **Sink rule:** the swap builder measures the DRIVABLE asphalt/rut surface height in Blender and sinks each tile so that surface is flush with terrain Y=0 — otherwise cars sit 0.35 m sunk into the road. Verify in-game that cars sit ON the asphalt. Keepers: `preview/models/road-straight.glb` 373,504 B, `preview/models/dirt-road.glb` 421,200 B. |
| **Ground clutter** (NEW — dressing only) | Grab3D bush (11,570 tris, ~1.1 m) · boulder-rock-set (10,112 tris) · small-rock-set (6,656 tris) · tree-stump (8,264 tris) — all CC0, grounded Y=0, §2a(ii) | **One embedded const per type; scattered as clones.** Placement rules: keep ≥2 m clear of the road centreline, building doorways, supply POIs and the checkpoint path; cluster around trees/buildings/field edges, never in a neat grid. NO collision in this build (dressing only — walking through a small rock is accepted; boulders are small sets, not walls). Verify scatter density in a top-down screenshot — sparse enough to read as a wasteland, not a rock garden. **Keepers gated PASS 2026-11-12 ~00:35 AST (coordinator-verified):** bush 340,440 B, boulders 297,940 B, rocks 283,928 B, stump 295,676 B. Extras total 2,780,024 B GLB ≈ 3.7 MB embedded. |

### 3c. Keep as-is (no swap)
- `u6`/`P.scene` car (sedan) — **no longer the chase car.** The Jeep is the car Kes finds
  and drives in the chase (Kester, 2026-11-11). The sedan stays in the world as the SECOND
  drivable in free roam (it has the interior + steering-wheel node the first-person view
  needs), and it is the natural villain ramming car in the chase stage (Appendix A) — the
  pirates' beater. Do not remove it; its role changed, not its existence.
- `J6`/`v` trees (base — extended, not replaced).
- `p6`/`g` house — **trace its in-world usage before touching it.** The cottages are
  procedurally built; the house GLB's placement is not yet mapped. Do not swap blind.
- `F6`/`j` cat — no cat replacement was ordered. Keep.
- `H6`/`B` cottage — **trace its usage before touching it** (same reason as the house).
- `f6`/`U` barrel, `X6`/`E` crate, `PS`/`y` far tree,
  `iS`/`T` shed, `TS`/`s` fire (campfire with animations — already works). (Pine `$6`/`n` and
  dead tree `AS`/`b` were here — both now have Grab3D replacements, §3a.)

**Residual boxy list (known + accepted in this build — the doc does NOT claim the swap fixes
"boxy" for these):** mob weapons (cleaver/bat/pipe/crowbar stay primitive — no CGTrader
mob-weapon models were ordered); cat, barrel, crate, shed, house, cottage, far tree stay
as-is. Kester is told plainly: these are the accepted remainder, not an oversight.

---

## 4. Enterable buildings plan

**Principle:** an enterable building = (1) a doorway cut in the wall, (2) interior wall colliders
with a gap at the doorway (the world's collider system is `Box3` lists — split each wall into
segments around the door), (3) an interior floor + walls the player can see from inside —
CGTrader exteriors are typically single-sided, so the builder uses **DoubleSide materials**
on the interior wall materials (verified from inside in the preview) or a separate interior
shell, with the cost/verification noted — no backface-culling tricks, (4) a `shelters` rect
entry so `inShelter()` recognises it (warmth/health recover, mob will not enter) — the rect
is measured from the ACTUAL interior footprint in the preview, NOT copied `w/2-.35` from the
cottages (new buildings have unknown wall thicknesses; the cottage margin may put the rect
through the walls), and **shelter applies ONLY while the door is shut** — if the door stands
open, the shelter buff stops and mobs may enter (the glow rule's whole point is that a shut
door is the boundary), (5) a door mesh with a knob that glows on approach (§5), (6)
furniture/props inside so the interior isn't an empty box. **Interior lighting: flashlight
only** — no interior lights; on Days 1–2 daylight comes through the doorway/windows, on
Night 3 the player's flashlight is the light source.

**Placements — every placed object gets (x, z, rotation) + a clearance rule, verified against
the bundle's collider/POI/marker lists before §9 step 8:**

| Object | Placement (x, z, rotation) | Clearance rule |
|---|---|---|
| Medieval house | (-30, -44), facing east (door toward the road) | ≥10 m from the Alley Fire Site marker (-21, -44, radius 3.4); ≥8 m from the road centreline (x=0); ≥5 m from any cottage / supply POI. |
| Windmill | (30, -60), facing west (door toward the road) | ≥10 m from the Field Lab marker (15.8, -44); ≥8 m from the road centreline; ≥5 m from any cottage / POI. NO ladder to an upper floor in this build — single floor only (the ladder is deleted from the plan: `ladderAt`/`ladders` registration, upper-floor collider, and `inShelter()` returning false above 3 m are all unscoped). |
| Small village (4 homes) | Cluster anchor (-35, -70); homes spread ±15 m around it, doors facing the cluster centre | ≥15 m from the road centreline (x=0) — the cluster sits OFF the route to the checkpoint, not on it; ≥10 m from the checkpoint approach corridor; ≥5 m between homes; ≥5 m from any POI/marker. |
| Military Truck | (0, -80), angled ~30° across the main road | Full-width vehicle block with a ≥1.5 m pedestrian gap on the WEST side. The last ~20 m to the checkpoint is always on foot (see §3b). |

**Placement verification (§9):** a headless top-down screenshot OR a collider dump showing no
overlaps between the new placements and the existing cottages, 11 supply POIs, 8 decision
markers (radius 3.4), ammo-cache exclusion zones, the road, and the extraction route to the
checkpoint (0, -102). A placement that overlaps is moved before its swap commits.

**Known difference (accepted):** the procedural cottages keep their position-zone shelter
while the new buildings have real doors. The player can stand outside a solid cottage wall
and regenerate — this is the existing fiction, unchanged. A cottage-doorway pass is a later
build, not this one. The cottage GLB (`H6`/`B.scene`) is traced before §9 step 8 and stays
as-is; it is NOT an enterable building in this build.

| Building | Enterable? | Interior state | Plan |
|---|---|---|---|
| Medieval house | ✅ YES (page: "fully ENTERABLE") | EMPTY — "waiting for you to add your props" | Enterables, 75,596 polys, 3 LODs. Cut doorway (verify the source model has a door opening; if the wall is solid, cut it in Blender). Furnish: table + bed + crates. Furniture source: first check whether the small-village file has separable furniture meshes — if yes, reuse them; if no, furnish from the in-game prop library (crate/barrel) + a simple table/bed built from the house's own plank textures. Flag as "furnish pass 2" if the first pass is thin. Place at (-30, -44) — a mission interior for Day 2/3 (near the Alley Fire Site / The Field Lab). |
| Windmill | ✅ YES (page: "fully ENTERABLE") | EMPTY | Enterables, 10,868 polys, 3 LODs. Same as the house, lighter furnishing (sacks, barrels). Single floor only — NO ladder to an upper floor in this build (see the placements table). Place at (30, -60) as a landmark + mission interior on the village edge. |
| Small village (4 homes) | ✅ 3 of 4 (2 furnished, 2 empty — 1 closed, 1 broken door) | 2 furnished (ready), 1 empty with broken door (furnish lightly — the broken door is an open entry), 1 closed (stays shut) | Place as a distinct second village cluster, anchor (-35, -70), homes spread ±15 m, ≥15 m off the road centreline (see the placements table). The 2 furnished homes are ready-made mission interiors. The Day 3 shelter decision ("Where to Sleep") marker stays where it is — the village homes grant shelter buffs through their own `inShelter()` rects but do NOT move the decision (the decision triggers at the fixed Cellar Hatch marker (17.8, -38.5, radius 3.4); adding shelter rects elsewhere doesn't "wire" anything to it). The cluster is ~60 m+ from the hatch, in the dark, on Night 3 — the homes are usable shelter that night, the decision itself stays at the hatch. The CLOSED home is the fourth home, north-east in the cluster, boarded door: no glow, no entry, solid collider. E on it gives a "Locked — boarded shut" toast (a player who presses E and gets nothing reads it as broken, not deliberate). Its knob is present but NEVER emissive. |
| New York buildings (= L_build_1/2/3) | ❌ filler only | — | Skyline/backdrop. No doorway, no interior, no glow. The 3 GLBs are named L_build_1/2/3 inside the files — same models, one row. |
| Medieval block (Lumion scene) | ❌ DROPPED | — | `.ls11` is a proprietary Lumion format Blender cannot import; no Lumion available. Cannot be a backdrop either (no extractable geometry). DROP for this build. |
| Archviz scene | ❌ convert-or-drop | — | 514 MB Blender scene (`.blend`, proside, Royalty Free). The builder opens it in Blender; if exterior building meshes extract cleanly at reasonable poly counts, use 1–2 as skyline/backdrop fillers; otherwise DROP. |
| Existing cottages | Zone-only today | — | UNCHANGED in this build. Their "inside" stays a position zone. Do not attempt to cut doorways into the procedural cottages in this build — the new enterable buildings are the mission interiors; the cottages remain as they are. |

**Blender inspection gate (before §9 step 8 — go/no-go per building):** for each enterable
building, verify in Blender BEFORE any swap work: (a) a door opening exists (or a door mesh
that can be opened); (b) interior faces exist — the mesh is not a single watertight shell
with no inside (if it is, cutting a doorway is non-trivial: it breaks UVs/normals and needs
wall thickness — the builder reports it and the building is demoted to filler rather than
faked); (c) wall meshes are separable enough to split colliders around the doorway. A
building that fails any of (a)–(c) is a NO-GO for enterability in this build — it becomes
filler/backdrop, and the README says so. "Fully ENTERABLE" on a store page is a claim, not
evidence — the gate is the evidence.

**Mob exclusion — verify, don't assume:** whether mobs respect the world's `blocked()`
colliders is UNVERIFIED. Before claiming "mob will not enter", the builder checks the mob
movement code: if mobs call `blocked()`, the shut door + wall colliders already exclude
them; if they don't, the builder adds a doorway/interior exclusion (mobs treat shut doors
and interior rects as blocked) — or the doc drops the "mob will not enter" claim for that
building and the README says so.

**Door-knob glow + door interaction (§5) applies ONLY to the three enterable sets above**
(medieval house, windmill, village homes). Every other building gets no glow and no door.

**Collision note:** the world's `blocked(x, z, r)` checks `Box3` colliders. For each enterable
building the builder registers the wall segments (with the doorway gap) as colliders AND the
door itself as a collider while shut; when the player opens the door (E), the door collider is
removed/swung aside and the interior becomes walkable. The `inShelter()` rect for each building
is its interior footprint (w/2-.35, d/2-.35, same margins as the cottages).

---

## 5. Door-knob glow (Kester's rule)

**Rule:** when the player comes close to an enterable space that has a shut door, the door
knob glows — so the player knows that door can be opened and entered.

**Implementation (design, not code):**
- Each enterable building gets a door assembly: a door panel mesh + a knob mesh (small
  sphere/cylinder, **≥6 cm diameter** — a 3 cm knob at 3 m in the dark may be sub-pixel)
  parented to the door, on the outside face, at handle height (~1 m), PLUS a knob/handle
  on the INSIDE face so closing from inside works (glow + hint on the inside face when
  within 2 m). If the source model has no separate door mesh, the builder adds a simple
  panel + knob in Blender at the doorway (wood material, matches the building).
- **Hinge pivot:** rotating a door panel mesh rotates about its CENTRE, not a hinge. The
  builder creates a hinge pivot in Blender (pivot at the hinge edge; the door panel is
  re-parented to it) and the door swings on that pivot (~100° over 0.4 s). Swing
  direction: INWARD (into the building), hinge on the left as seen from outside.
  **Blocked-doorway rule:** if the player stands in the swing arc/doorway when closing,
  the door refuses to close and shows a "move clear" hint — it never pushes the player.
- **Knob material:** the knob gets its OWN cloned material instance per door. Setting
  emissive on a shared material would tint the whole door/building.
- **Per-frame hook:** `nearestInteractable()` does NOT run per frame — it is a query called
  from `tryInteract()`/`pushHud()`. The glow needs a per-frame tick: use the world's
  per-frame closures (the `i.push((d,u)=>…)` pattern) as the update owner, and integrate
  with the existing hint/prompt display. For each shut enterable door, if the player is
  within **4 m** of the knob, set the knob material emissive to warm gold (`#ffb347`,
  emissiveIntensity pulsing 0.6–1.2 with a slow sine) and show the hint "E — open door"
  when within 2 m. Beyond 4 m the knob is its normal material (no glow, no light — cheap:
  emissive only, NO extra point light). **Interaction radius: ONE radius** — glow at 4 m,
  hint + open at 2 m.
- **E-key slot:** doors are a FIFTH interactable with an explicit slot in BOTH
  `tryInteract()` and the `pushHud()` prompt chain: driving → `nearCar()` → DOOR →
  `nearestSupply()` → `nearestInteractable()`. A supply POI next to a door no longer
  shadows it, and a decision marker next to a door no longer shadows it. While driving, E
  near a door does nothing (you cannot open a door from a vehicle).
- On open (E): the door swings on its hinge pivot, the glow stops, the door collider is
  removed, and the interior is walkable. **Collider hook (verify first):** `colliders` is
  a plain Box3 array — verify `blocked()` live-reads that array (vs a prebuilt structure)
  before relying on dynamic removal. If it doesn't, the door collider is a separate entry
  the builder toggles, or the doorway gap stays permanently open in the collider set with
  the SHUT door itself as the collider. On close (E again from inside): the door swings
  shut and the collider returns. The closed village home NEVER glows and NEVER opens.
- The glow must also be visible at night (stage 3) — emissive is unlit, so it reads in the
  dark. This is the point of the feature on Night 3.
- Verification: headless screenshot at 3 m from a shut door shows a visible gold knob;
  at 6 m it does not. Headless positioning: scripted camera pass or a debug teleport query
  param (the doc names the method — no harness, no verification). Live check on the Pages
  preview by walking up to a door. **§8 gate: E at a door next to a car opens the door,
  not the car.**
- **Multi-door rule:** if more than one shut enterable door is in range, ONLY the nearest
  glows — one glow at a time, so the rule stays readable.
- **Knob visibility criterion:** in the 3 m headless screenshot the knob must be ≥8 px
  across and gold-distinct from the door panel; at 6 m it does not glow.

---

## 6. Progression + flow improvements

Verified gaps in the current flow (from reading the bundle) and the fix for each.
Scope: implement items 1, 2, 4, 6, 7, 9, 12, 13, 14 in this build. Items 3, 5, 8, 10 are
verify-only — each with a named check (owner: coordinator): item 3 — observe the HUD stat
bars in a headless playthrough screenshot set immediately after choosing (bars must move
within 1 frame of the choice); item 5 — verify the yellow-flashlight viewmodel is in hand
on Night 3 in a first-person screenshot; item 8 — verify fire damage is 16/sec in a
headless test (stand in fire 1 s, health −16); item 10 — verify the hen sound plays for the
new Hen model (kind `"chicken"` → hen/rooster MP3) in a headless audio check. A verify-only
item with no method is skipped silently — the check IS the method. Item 11 rides on the
Worker/stranger swap (§3b).

1. **Dead time between decisions (HIGH).** Each stage is: walk to marker A, walk to marker B,
   walk to the checkpoint — across a ~406 m map on foot at ~4–5 m/s (the Day 1 route alone
   is ≈381 m: spawn (0,264) → Well (7,48) → Pantry (-14.8,38) → checkpoint (0,-102) =
   76–95 s of pure walking, before mobs or detours), with nothing to do in between except
   wandering animals and periodic mobs. **The pickup system ALREADY EXISTS — this item is a
   modification, not a new system.** `nearestSupply()`/`takeSupply()` are already wired
   into `tryInteract()` (priority: driving → `nearCar()` → `nearestSupply()` →
   `nearestInteractable()`), fed by 11 FIXED `world.pois` (water jugs e.g. (-9,52), (7,-20),
   (9,232); ration boxes e.g. (9,66.5), (-8,-12)) giving **+35 thirst / +30 hunger**
   (NOT +20/+20), radius 2.4, already shown on the map via `sitesOf("water"/"food")`.
   `advanceStage()` never resets them — it only calls `spawnAmmo()` and `mobs.clearAll()` —
   so they are ONE-SHOT for the whole game, not per stage. **Fix:** (a) restock all 11 POIs
   in `advanceStage()` (per stage), keep the +35/+30 values (no reason to nerf them); (b)
   add BANDAGES as a genuinely new pickup kind (+10 health) with its own `nearestSupply`
   kind, a HEALTH map category, fixed waypoints between the decision markers, and an
   `advanceStage()` reset — bandages do not despawn within a stage. The Military Truck bed
   is one bandage pickup site (reach-from-ground radius 2 m at the tailgate, §3b).
   **Inventory first:** the builder counts the existing `world.pois` (count, type, position,
   stage) at build start — the 5 positions above are examples; the remaining 6 come from
   the inventory. **Trade-off (standing principle):** every pickup also makes the player a
   bigger target — mob spawn pressure scales with how much the player is carrying (see the
   principle at the top of this doc). Picking up supplies is a choice between having more
   and being more of a target, not a free win.
2. **No guidance on where to go (HIGH).** Markers exist but the player must open the map (M)
   to find them; a first-time player wanders. **Fix:** an objective compass arrow (DOM
   overlay at the screen edge) pointing to the nearest incomplete decision marker, with a
   distance readout in metres. **Extraction phase:** once both stage decisions are done,
   there is no incomplete marker — the arrow switches to the green flare/checkpoint
   (0, -102). **While driving** the arrow stays visible (the player most needs it then).
   Keep the map as the detail view. Reuse the existing marker positions — no new data needed.
3. **Decision stakes are abstract (verify-only).** Choices are picked blind (rank hidden until
   after) and the lesson shows after — that is the design, keep it. Verify the HUD stat bars
   animate immediately on choosing, so the consequence is visible. If they don't, wire it.
4. **Day 4 is thin (MEDIUM).** Radio + signal, then an empty walk to the checkpoint = win.
   The helicopter decision is the climax but nothing pushes back. **Fix:** a final mob surge
   on Day 4, built as a SEPARATE one-shot spawn (not the normal spawner). **Trigger** — the
   moment extraction goes active on Day 4 (the checkpoint toast / flare becoming visible —
   name the exact code hook at build time). **Spawn** — one group (4–6, mixed bodies)
   placed DIRECTLY on the road between the player's position and the checkpoint, bypassing
   the normal spawner's rules (the normal spawner only counts down while zero mobs are
   alive, only picks building corners 42–95 m away, and `advanceStage()` calls
   `mobs.clearAll()` with `spawnT=40` — none of that applies to the surge; it is a direct
   `spawnGroup()` call with explicit positions). **Provoked on spawn** — the surge group is
   marked provoked (exempt from `unprovoked()` −10/−25), or the climax punishes the player
   for fighting it. **On foot, by design:** the Military Truck roadblock at (0, -80) stops
   the car — the player MUST continue on foot from the truck to the checkpoint (~20 m) —
   so the surge is a genuine on-foot fight, not a drive-by (`hitByCar()` kills any mob at
   speed >4 for 200 damage, and the checkpoint win triggers on proximity within 7 m — if
   the player could drive through, the "final fight" would be trivial). Toast: "One last
   push — they are between you and the flare." The walk to the checkpoint becomes the final
   fight-or-flight.
5. **Night 3 is the only night (no change).** Flashlight matters most here. No flow change;
   the yellow-flashlight viewmodel swap (§3b) lands globally, which is the improvement.
6. **Vehicles are undiscoverable (MEDIUM) — per-car, not blanket.** There are THREE cars
   already, and they are not equally hidden. The station wagon (6.5, 250) sits ~15 m from
   the Day-1 spawn (0, 264) — the player can hardly miss it, so it gets NO discovery toast
   (it is the tutorial car). The sedan (6.5, -62) is the remote one. **Fix (per vehicle):**
   once the player comes within 30 m of the sedan, the Jeep, or the bike, add a map POI
   ("Car", "Jeep", "Bike") and a one-time toast ("A working car — E to drive" / "A working
   Jeep — E to drive" / "A working bike — E to drive"). The wagon gets a map POI but no
   toast. The Jeep is the FOURTH drivable — its reason to exist beyond "second" is that it
   is the car Kes finds and drives in the chase (Appendix A).
7. **Fall damage is softer than the bar (SMALL).** Bar: ~half health from a short building,
   lethal ~12 m. Code: (A-2.5)*9 → 5 m = 22, 6 m = 31. **Fix (piecewise — a single linear
   formula cannot hit the bar):** `damage = A<=3 ? 0 : A<=6 ? (A-3)*16 : 48+(A-6)*8.7`
   (0 at 3 m — the bar says none under 3 m, and `fallDamage` only fires for falls >3 m;
   5 m = 32, 6 m = 48 ≈ half health; 10 m = 83; 12 m = 100 = death). The earlier (A-3)*16
   draft was WRONG: it reaches 100 at ~9.25 m, silently lowering the lethal threshold from
   12 m. The piecewise curve keeps the effective lethal height at 12 m (the hard kill at
   A≥12 still applies). Lethal threshold stays 12 m.
8. **Fire damage is correct (no change).** 16/sec matches the bar exactly.
9. **Fighting mobs has no reward (MEDIUM).** Mobs cost ammo/health and give nothing back.
   **Fix:** 25% chance PER KILL that a dead mob drops a bandage (+10 health) or +6 shells.
   The drop is a NEW entity type, not attached to the corpse (dead mobs sink after 14 s and
   are removed at `deadT>18` — a drop on the corpse would vanish with it). Lifetime: 60 s
   or until the stage ends, whichever comes first. Pickup radius 2 m, its own
   `nearestSupply` kind ("drop"), E-priority in the supply slot (below doors, §5). Drops
   are cleared in `advanceStage()` (which clears only `ammoBoxes` and mobs today — drops
   would otherwise persist across stages and accumulate). No map marker — you see the glow
   on the ground. Reward note: +6 shells (not 3 — ammo comes in +18 boxes against a
   60-round cap, so 3 is trivial) or a bandage (+10 health).
10. **Animal encounters are ambient (verify-only).** Chickens/cows/dogs wander; feral ones
    attack. Verify the hen sound plays for the new Hen model (kind stays `"chicken"` → the
    hen/rooster MP3 is already wired). No new animal mechanics in this build.
11. **The Stranger is the emotional peak (HIGH, via swap).** The Day 2 stranger scene gets
    the Worker model in a seated pose (§3b). No extra flow change — the realistic figure IS
    the improvement.
12. **Cold open (SMALL, high value).** The game starts at spawn with intro text and no
    objective. **Fix:** a one-time objective toast at the start of Day 1: "Day 1 — find
    water and food. Press M for the map." (And the compass arrow from item 2 does the rest.)
13. **Fight or flight on foot (HIGH — standing rule, Kester 2026-11-11).** When attacked on
    foot — by people (mob, NPCs) or animals (dog, any hostile animal) — the player has two
    options: fight and kill, or run until the attacker RESETS. **Implementation:** each
    attacker has a reset radius (tuned in playtesting, start 40 m from its spawn/aggro
    point): if the player gets farther than that, the attacker visibly gives up — stops
    chasing, turns, and walks back to where it came from (the reset must be READABLE: a
    visible give-up + return, not a silent despawn). Fleeing costs nothing but the threat
    stays for next time; fighting costs health/ammo but removes the threat permanently.
    The reset applies to mobs AND feral animals (dog 40% / cow 30% feral) alike. Does NOT
    apply inside a vehicle (the chase stage is the in-vehicle exception — Appendix A).
14. **Engine + brake sound (MEDIUM — Kester's bar).** Engine and fire sounds are
    synthesized today; the bar wants a proper engine-acceleration sound, and brake screech
    is in the README bar. **Fix:** source a recorded engine loop + brake screech (free,
    license-checked — same license rule as §2), wire the engine loop's pitch/volume to
    speed, screech on hard braking (SPACE at speed). Verify by ear in the headless
    playthrough + on the live preview. If no suitable free recording is found, state
    plainly that the synthesized engine ships and why — do not fake it with a louder synth.

**What is NOT changing:** the 4-stage structure, the 8 decisions and their text, the stat
system, the death/win conditions, the daylight-only Stage 1 rule, the night-only Stage 3
rule, the start-screen text and orange frosted glass, the decision pop-up styling.

---

## 7. Format conversion plan (FBX / .blend / OBJ → GLB)

**Tooling:** Blender 4.5.0 portable (`~/workspace/choices/work/blender-4.5.0-linux-x64.tar.xz` —
download COMPLETED 2026-11-11 ~19:53, 376,170,648 bytes, verified healthy via `xz --list`:
358.7 MiB compressed, 1120 MiB uncompressed, CRC64 clean. The Goldfish review's "corrupt
tarball" finding was true at check time and is now stale — the download finished after the
check. Extract to `work/blender/`.) Headless:
`blender --background --python convert.py -- <model> <out>`. Blender was NOT installed on
the VM; the portable tarball is the install (free, no apt needed — apt is unusable on this
VM: `apt-get update` hung 9+ minutes and the package lists are empty).
**Fallback if Blender fails:** `pip install pygltflib` + manual GLB assembly is NOT a real
fallback for FBX — Blender is the conversion path. If Blender cannot run (RAM is 7 GB —
convert ONE model at a time, close Blender between conversions), the build stops and
reports; do not hand-roll an FBX parser.

**Per-model conversion script** (`work/convert.py`, one model at a time — **step-1 deliverable:
it does not exist yet; the builder writes and tests it in §9 step 1**):
1. Import the source (FBX / .blend / OBJ). For `.blend` (Hen), open the file and export the
   `hen` object directly (the .blend is the author's working file — verify it contains the
   same mesh as the FBX; prefer the FBX unless the .blend is cleaner; verify the .blend's
   textures are PACKED before choosing it — it may reference unpacked external textures).
2. **Unit-scale normalization.** FBX sources are commonly centimetre-scale. Normalize to
   metres in Blender (1 unit = 1 m) — only `cg()` normalizes scale in-game, and mob/human
   models never pass through `cg()`, so nothing else normalizes them except the preview
   eyeball. Verify the exported model's bounding box in metres before export.
3. Apply transforms: FBX imports Z-up → convert to Y-up on export (Blender's glTF exporter
   handles this; VERIFY the exported model stands upright and faces +Z in the preview).
   **Check for unapplied −90° armature rotation and unapplied transforms** — apply all
   transforms before export (the real axis risks, not the importer's conversion).
4. Adapter renames per §3 — **BONE renames, not Empty proxies** (materials: `feather` /
   `hide` / `patch`; bones: `legL`/`legR`/`head`, `legFL/FR/BL/BR`,
   `legL/legR/armL/armR/torso/head` for mobs; gun `muzzle` Empty — an Empty is fine as a
   static anchor). Record the per-body bone→slot mapping table.
5. Decimate to the poly AND vertex budgets (§7 budgets) with Blender's Decimate modifier
   (collapse, preserve UVs). **For skinned/multi-object models: decimate PER OBJECT (the
   Norton is 119 objects), with the Armature modifier AFTER the Decimate modifier in the
   stack, and preserve vertex groups** — decimate-vs-Armature order is load-bearing. Check
   the silhouette still reads after decimation — never decimate blind.
6. Textures: resize to max 1024 px (2048 for hero vehicles) — **mechanism:** `Image.scale()`
   in the script, re-save as JPEG unless alpha is needed, re-pack into the file before
   export; the glTF exporter embeds the packed textures.
7. **Interior faces (enterable buildings only):** set DoubleSide on the interior wall
   materials (or build a separate interior shell) — downloaded exteriors are single-sided;
   without this the inside is invisible. Verify from inside in the preview.
8. **Animation pipeline:** keep baked animation clips with clear names (NLA/action step in
   the script) — BUT the code has no AnimationMixer for mobs (mixers exist only for
   birds/fire), so a baked mob walk clip is a NOTE, not the plan; the procedural walk
   (§3b) is the default and no mixer is added unless the coordinator approves it.
9. Export glTF 2.0 binary (.glb), +Y up, embed textures. Record: vertices, faces, texture
   count/size, animations, file size.
10. Write a conversion report line to `work/conversions.md` (model, source file, poly/verts
    before/after, texture sizes, GLB size, adapter renames applied, peak RSS of the
    conversion).

**Size budgets (base64 inflates GLB by ~33% — the table shows BOTH columns; the 32 MB cap is
on the FINAL `index.html` BYTES, not on GLB bytes):**

| Model | GLB budget | Embedded (base64) | Notes |
|---|---|---|---|
| Each human (Worker / Woman / Zane) | ≤ 1.5 MB | ≤ 2.0 MB | **REVISED 2026-11-11 ~20:45 AST (builder measurement):** the 0.9 MB cap is unachievable — geometry alone at ~16–18k verts ≈ 0.9–1.1 MB. Measured floors: Worker 1.24 MB (16,319 verts), Zane 1.82 MB (14,367 verts). Zane must fit 1.5 MB via 512 px textures + further decimation; if he can't, DROP HIM (per §10 item 2). Woman MEASURED 2026-11-12 ~00:30 AST: keeper 1,140,196 B, 16,818 verts — inside budget (§2a(i)). |
| Hen | ≤ 400 KB | ≤ 0.53 MB | Source is 418 KB FBX — should fit easily. |
| Cow / Dog / Bird (when they land) | ≤ 600 / 600 / 300 KB | ≤ 0.8 / 0.8 / 0.4 MB | |
| Honey Locust (1 variant) | ≤ 600 KB | ≤ 0.8 MB | One const, referenced twice (oak + 4th tree variant) — counted ONCE. |
| Each grass species (2 max) | ≤ 250 KB | ≤ 0.33 MB | 2 species max (was 3). |
| Grab3D Pine | ≤ 400 KB | ≤ 0.53 MB | 8,032 tris, 5.7 m — already light; mostly texture compression. Replaces the pine slot (old GLB ~40 KB freed). |
| Grab3D Dead Tree | ≤ 400 KB | ≤ 0.53 MB | 13,849 tris, 4.5 m. Replaces the dead-tree slot. |
| Road tile (each type) | ≤ 500 KB | ≤ 0.67 MB | Straight road source is 5.5 MB (heavy textures) — compress to ≤512 px. Dirt road 24,551 tris. One const per type, instanced along the road. |
| Ground clutter (each type) | ≤ 350 KB | ≤ 0.47 MB | Bush 11,570 tris / boulders 10,112 / rocks 6,656 / stump 8,264. One const per type, scattered clones. |
| Medieval house | ≤ 1.0 MB | ≤ 1.33 MB | |
| Windmill | ≤ 800 KB | ≤ 1.07 MB | |
| Small village (4 homes) | ≤ 1.0 MB | ≤ 1.33 MB | If the village ships as 4 separate GLBs, the 1.0 MB budget is the TOTAL for all four. |
| Norton Manx | ≤ 1.0 MB | ≤ 1.33 MB | 404k faces → ≤50k faces first. |
| Jeep Renegade | ≤ 1.0 MB | ≤ 1.33 MB | Low-poly source — should fit. |
| Old Car 04 | ≤ 1.0 MB | ≤ 1.33 MB | |
| Military Truck | ≤ 1.0 MB | ≤ 1.33 MB | |
| Shotgun / Machete / Flashlight | ≤ 200 KB each | ≤ 0.27 MB each | |
| **CORE TOTAL added (GLB)** | **≤ ~14.9 MB** | **≤ ~19.9 MB embedded** | Humans revised 2.7 → 4.5 MB GLB (see the human row above). |
| Less: removed GLBs (chicken 154 + cow 357 + dog 116 + bird 185 + wreck 87 + oak 43 KB ≈ 0.92 MB GLB ≈ 1.23 MB base64 freed) | | **−1.23 MB** | |
| **NET added to `index.html`** | | **≤ ~18.6 MB** | Current file 14.4 MB → target total **~33.0 MB — OVER the 32 MB hard cap by ~1 MB.** With the Grab3D extras (§2a(ii): 8 new consts ≈ +4.2 MB embedded) the projection is **~37 MB — OVER by ~5 MB.** The cap stands: the running total (tracked in §11) decides at each swap, and the cut order below applies — the extras are the newest pure additions and are cut as a group (after fillers, before Zane) if the total projects over 32 MB. Pine/dead-tree swaps are NOT in the cut group (they replace existing slots at near-zero net cost). |
| Fillers — ONLY if budget remains: NY skyline (= L_build_1/2/3 GLBs) ≤ 1.0 MB GLB total; archviz fillers ≤ 0.5 MB GLB total (only if the `.blend` extraction is clean) | | ≤ 2.0 MB embedded | These ship ONLY if the running total (tracked in the swap log, §11) stays ≤ 32 MB after adding them. They are the first cut. |

**Vertex budgets (Goldfish — a byte budget alone doesn't constrain geometry):** the Woman at
163,993 verts is ~10 MB of geometry alone (position + normal + UV + skin indices/weights ≈
64 bytes/vert) — she cannot fit a 1.5 MB GLB without decimating to roughly a tenth, which
the v1 doc never stated for her (it stated it for the cartoon Girl). Budgets: each human ≤18k verts (Woman 164k → ~16k; the cartoon Girl 48k → ~16k, already proven at 16,515 verts / 1,483,116 B — the fallback keeper); Hen ≤3k; cow ≤8k; dog ≤6k; bird ≤3k; Honey
Locust ≤10k; each grass species ≤2k per plant; each enterable building ≤30k; Norton ≤50k
faces (already); Jeep ≤20k; Old Car ≤20k; Military Truck ≤25k; each weapon ≤2k. Grab3D
extras — already light, texture compression is the work, not decimation: pine ≤8k verts
(as-is 8,032 tris); dead tree ≤14k; dirt road ≤25k; bush ≤12k; boulders ≤11k; rocks ≤7k;
stump ≤9k. Zane's
decimation target: ≤18k verts from the 109 MB source (32,771 polys). Worker: 34,990 tris
≈ ~17–18k verts — likely fits with light decimation; state the exact count from the
Blender inspection at §9 step 1. **Source poly counts (verified 2026-11-11 ~20:30 AST):**
Worker 34,990 tris · Woman 163,993 verts / ~300,000 tris (rigged GLB; static version 305,702 tris) · Girl 70,497 polys (71k version; 48k version also in the file — FALLBACK ONLY) ·
Zane 32,771 polys · Hen 1,062 polys · cow 3,814 polys · bird 22,905 polys · Honey Locust
1,125,789 polys total (6 variants) · Norton 404,291 polys · Jeep 4,734 polys · Old Car
295,000 polys · medieval house 75,596 polys · windmill 10,868 polys · shotgun 6,160 polys
· machete 989 polys · flashlight 2,060 polys · Grab3D pine 8,032 tris · dead tree 13,849 ·
stump 8,264 · straight road 4,220 · dirt road 24,551 · bush 11,570 · boulders 10,112 ·
rocks 6,656. **Format paths:** FBX is the main path;
BLEND sources (Norton, Military Truck, archviz) open directly in Blender — prefer the
BLEND for the Norton (may have better rigging) and the Truck (the 16 KB GLB is too
low-detail to use directly); the shotgun is ALREADY GLB (no conversion — just
decimate/resize/verify); the medieval block (Lumion `.ls11`) has NO path — DROPPED.

**Why the old table was wrong (Goldfish):** it summed to ~23.1 MB of GLB (≈30.8 MB base64)
against an "18 MB" total that never said GLB or base64 — 14.4 + 30.8 ≈ 45 MB, over the cap
by ~13 MB. The table above is the honest math. **Cut order if the running total approaches
32 MB:** (1) texture resolution, (2) NY skyline / archviz fillers, (3) road + ground clutter
as a group (pure additions — the pine/dead-tree swaps stay, they replace existing slots),
(4) Zane — drop him entirely, (5) one grass species. Never silently exceed 32 MB — report it.

---

## 8. Preview-before-commit procedure (Kester's rule)

**Viewer:** build `preview/model-preview.html` on the character-preview pattern:
single HTML file, **vendored** Three.js (the VM's headless Chrome cannot reach public CDNs —
`preview/vendor/` already holds `three.module.js` + `controls/OrbitControls.js`; ALSO vendor
`GLTFLoader.js` from three 0.160.0 examples into `preview/vendor/loaders/GLTFLoader.js` —
the character-preview maps `"three/addons/"` → `"./vendor/"`, so the loader MUST land at
`vendor/loaders/GLTFLoader.js` with importmap entry
`"three/addons/loaders/GLTFLoader.js": "./vendor/loaders/GLTFLoader.js"` — any other path
404s). The viewer takes a `?model=<name>.glb` parameter, loads the GLB, and shows it
on a 1 m grid with a 1.8 m human silhouette for scale, orbit controls, and buttons for
Idle / Walk / Run where the model has animations. Publish it to GitHub Pages with the repo
so Kester can open any model preview himself.
**Viewer smoke test (before the first model preview):** load a known-good GLB extracted from
`index.html` (the chicken `q6`/`t.scene`, decoded to a file) — if the viewer can't show it,
fix the viewer before any model gate runs. A broken viewer blocks the build; it is not
skipped. **The game's own Three.js version is NOT identifiable from the bundle** (no
`REVISION` string in the minified code — verified by grep 2026-11-11). The preview uses
r160. If a model behaves differently in-game vs in the preview, suspect a version
difference first.

**Per-model gate (ALL must pass before the model is committed to the game):**
1. The GLB loads in the viewer with NO console errors.
2. Screenshot (headless Chromium, vendored libs) eyeballed by the builder: proportions
   natural (no elongation, no toy look), textures present and correct, no missing parts,
   no inside-out faces.
3. Scale check: the model measures correctly against the 1.8 m silhouette (human ~1.7–1.9 m,
   hen ~0.5 m, cow ~1.5 m at shoulder, car ~4.5 m long, door ~2.1 m high).
4. Orientation check: the model faces the game's forward axis (+Z) — verify by rotating it
   in the viewer and by the in-game screenshot.
5. Adapter check: required node/material names exist — for SKINNED models these are RENAMED
   BONES (`legL`/`legR`/`head`, `legFL/FR/BL/BR`, `legL/legR/armL/armR/torso/head` for mobs
   + the weapon socket), NOT Empty proxies (inert on skinned meshes); materials (`feather`,
   `hide`/`patch`); the gun `muzzle` Empty (a static anchor — fine). Verified by reading
   the GLB JSON, not by eye. For mobs: rotating each renamed bone in the viewer swings on
   the expected axis.
6. Animation check: any baked clip plays in the viewer (walk/run/flap/fire). Passes
   vacuously for models with no clips — a clipless model is judged by its still + the
   in-game screenshot (§9), and the bird must never ship frozen (see §3a).
7. Budget check: GLB size + poly count within §7 budgets (byte AND vertex budgets).
8. The preview page for that model is committed + pushed (so the preview link exists) AND
   `curl` returns HTTP 200 for the preview URL — pushed ≠ exists, check it.
9. **License gate:** source URL + license recorded in the README swap log BEFORE the commit
   — a model can pass gates 1–8 and still not ship with an unclear or non-commercial
   license. **Satisfied for the 23 CGTrader files by §2a** (author + license per file,
   verified by the parent 2026-11-11 ~20:30 AST) **AND the 11 Grab3D files by §2a(i)/(ii)**
   (all CC0 1.0, license files saved alongside each model); Maxtree's CUSTOM license is recorded
   verbatim. The dog is the only pending license.
10. **Enterable buildings only — FURNISHED:** an unfurnished interior does not ship
    ("furnish pass 2" means it does not ship in pass 1, not that it ships empty).
11. **Door gate (enterable buildings only):** E at a door next to a car opens the door, not
    the car (§5).

**A model that fails any gate does NOT go into `index.html`.** Fix it (re-convert, re-decimate,
rename) and re-preview. Never commit a model on the builder's word alone — the coordinator
checks the preview screenshots personally before the swap lands.

---

## 9. Build order (per-swap commits, live verification each time)

Work in this order. One swap = one commit + push + live check. If a swap's verification
fails, STOP that swap (do not stack more swaps on top of a broken build) and report.

1. **Tooling + spike tests:** tag the pre-build known-good commit FIRST (`git tag pre-integration-<date>` — the rollback anchor; see the rollback rule below). Blender extracted + verified (`blender --version`). **SPIKE-TEST THE LARGEST CONVERSIONS FIRST — Zane (109 MB FBX), then the Norton (404k faces, 119 objects) — before the Hen proves the pipeline.** Hen-first means the riskiest files are discovered late, after the pipeline is "proven" on the easiest file. Record peak RSS per model in `work/conversions.md`. **Abort rule:** if Zane cannot convert within RAM, drop Zane immediately per §10 item 2. The VM has ~7.9 GB total RAM and showed only ~344 MB available at the Goldfish check (other work running) — the builder checks free RAM and closes other work before starting Blender. `work/convert.py` written (step-1 deliverable). Blender inspection of Worker / Girl / Zane / Hen animation status (all four — the "unverified until Blender" rule, §3b) + Worker vertex count + Hen material→mesh mapping. **Flow interfaces defined BEFORE any placement work:** pickup kinds (POI restock, bandage kind, drop kind), HEALTH map category, discovery-toast rule (§6 items 1/6/9), surge trigger (§6 item 4), door E-slot (§5). `preview/model-preview.html` built + published + smoke-tested with the known-good chicken GLB (§8). **Builder findings 2026-11-11 ~20:40 AST (step 1 done — see INTEGRATION-STATUS.md):** Girl is UNRIGGED (rigged 2026-11-11 ~21:43, `work/rig_girl.py` — keeper exists as FALLBACK ONLY); **the female mob is now the Grab3D Woman Base Mesh (§2a(i)) — rigged with walk+run, conversion + walk gate in flight; do NOT hold step-4 on her: the pool ships with the female slot held as a clean swappable placeholder (Worker + Zane live) until her keeper passes the gate**; Worker export bug solved (fbx2gltf pipeline); Norton spike was blocked (file not landed — NOW LANDED, unblock it); per-human budget revised to 1.5 MB GLB (§7); license gate SATISFIED (§2a).
2. **Hen → chicken (`q6`/`t.scene`).** Smallest, safest first swap. Proves the pipeline end-to-end.
3. **Old Car 04 → wreck (`W6`/`c.scene`).** Simple static swap; also upgrades the stranger scene's backdrop.
4. **Worker + Woman + Zane → mobs.** The biggest visual win. Female slot = Grab3D Woman (§2a(i)); cartoon Girl is fallback only (see §10 item 3). Includes the `torso` proxies
   and procedural walk. Verify mob walk/chase/windup in a headless playthrough screenshot
   set (or a short scripted camera pass).
5. **Worker (seated) → Wounded Stranger.** Same model, baked seated pose.
6. **Shotgun / machete / flashlight → viewmodels** (when the files land). Verify the
   first-person screenshots (weapon in hand, muzzle position).
7. **Trees + road + clutter:** Honey Locust → oak (`_6`/`r.scene`) + 4th tree variant;
   Grab3D Pine → pine (`$6`/`n.scene`); Grab3D Dead Tree → dead tree (`AS`/`b.scene`);
   Maxtree grasses → instanced tufts. **Road + ground clutter (NEW, §3b):** straight asphalt
   tiles along the main drivable road, dirt tiles on village paths + the chase back-roads
   fork, bush/boulders/rocks/stump scattered per the §3b placement rules — one const per
   type, road path mapped from the terrain + landmarks first, top-down screenshot verifies
   placement BEFORE the road commit. The road/clutter commit is a separate per-swap commit
   AFTER the tree swaps, and it is the first cut if the running total (§11) projects over
   32 MB (see §7 cut order — the pine/dead-tree swaps stay; only the pure additions cut).
8. **Enterable buildings:** medieval house → windmill → village homes, each with doorway,
   colliders, shelter rect, door + knob + glow. Furnish pass per §4. Placement verification
   (top-down screenshot or collider dump, §4) BEFORE each building's swap commits. The
   truck-bed bandage pickup (§6 item 1) and the door E-slot (§5) are built with the
   buildings, not after them.
9. **Vehicles:** Jeep (the car Kes finds and drives — primary drivable + chase vehicle) →
   Norton (rideable) → Military Truck (roadblock + supply site). Vehicle POIs + discovery
   toasts (§6 item 6) are built with the vehicles, not in step 11.
10. **Fillers:** New York skyline (= L_build_1/2/3 GLBs); archviz fillers only if the
    `.blend` extraction is clean (§4). Medieval block DROPPED.
11. **Progression + flow items (§6) — the REMAINING ones:** compass arrow, Day 4 surge, fall
    rebalance, opening toast, fight-or-flight reset, engine sound. (Supply restock/bandages/
    drops and vehicle POIs were built with steps 8–9 — the flow interfaces were defined in
    step 1, before placement. These are code edits in `index.html` — small, testable one
    at a time.)
12. **The DOG slots into steps 2/7 when it lands tomorrow (first priority)** — the only
    pending file. All other swaps proceed per the §2b dependency table; a swap whose file
    hasn't landed is skipped with a clearly-marked README TODO. Do not hold the whole
    build for one file.

**Per-swap verification ("done" means ALL of these):**
- **License + source URL recorded in the §11 swap log BEFORE the commit (item 0).** No
  commit without them. **§9 item 0 is SATISFIED for the 23 landed files** — the browser
  task's final report IS the parent's inventory in §2a (author + license per file,
  verified on disk 2026-11-11 ~20:30 AST). The builder records source URL + license per
  swap in the §11 log from §2a and does not ship any model whose license is unclear.
  The dog is the only pending license.
  (Correction: "CGTrader free models are Royalty-Free unless the page says otherwise" is
  UNVERIFIED — CGTrader free models carry varying licenses, some editorial / attribution /
  non-commercial. Verify per model; do not assume. Maxtree is CUSTOM LICENSE, not
  Royalty Free — record it verbatim.)
- The new base64 const replaced the old one exactly once (or the new const was added once);
  the `data:model/gltf-binary;base64,` count is as expected (18 + new ones).
- **Post-embed check:** decode the base64 actually in `index.html` for the swapped const
  and compare node names + bounding box against the pre-verified GLB — this catches a
  wrong-const swap, which the base64 count check cannot (the count is unchanged by a
  wrong-const swap).
- Headless Chromium loads the local `index.html` with NO console errors and renders WebGL
  (screenshot shows the 3D scene, not a blank page).
- A headless in-game screenshot shows the new model in place, at the right scale and
  orientation. (The old "or the model preview" loophole is deleted — a swap can pass on the
  preview page while broken in-game. The §8 preview gate is separate and ALSO required.)
- For the mob swap (step 4): a headless frame-sequence capture of the mob walk/chase —
  the coordinator judges it; stiff = reject, bake a walk cycle (§10 item 1).
- Commit + push; poll the live Pages URL with a timeout (up to ~10 min, 30 s interval):
  GitHub Pages may gzip (Content-Length then differs from local size — compare the body
  hash instead) and Pages deploy lags the push (curl immediately after push can fetch the
  OLD file and "pass" with the old length — wait until the served bytes change). HTTP 200,
  correct title. Kester checks the live preview himself — the coordinator reports the live
  link per swap.
- README swap log updated (model, source URL, license, format, GLB size, pre-swap commit
  hash, live verification).

**Rollback (Goldfish):** the pre-build tag from step 1 is the known-good anchor. If a swap
passes headless checks, goes live, and breaks on Kester's phone, the owner (coordinator)
reverts THAT swap's commit (`git revert <swap commit>` + push + live re-verify) — never a
blind `git reset` on main, never stacked fixes on a broken build.

**Real-device / DeX check:** at minimum at the end of the build (and per major swap — mobs,
buildings, vehicles), the swap is checked on Kester's surface (DeX/phone): load time on a
phone network, fps with the new content, no touch/viewport breakage. Headless Chromium
renders WebGL in software (SwiftShader) — it cannot prove phone performance. Gate: the game
must hold a playable frame rate with 1,500–2,500 3D grass instances + new buildings on the
target device; if it doesn't, cut instance counts / texture resolution before shipping.

---

## 10. Risks + open questions

| # | Risk / question | Mitigation / owner |
|---|---|---|
| 1 | **Worker / Zane may have NO baked walk/run animations (the Woman HAS baked walk + run — §2a(i)).** The mob walk would then be fully procedural bone rotation, which can look stiff. | Verify in Blender FIRST (step 1 of the build) — all animation claims are UNVERIFIED until then. The plan is procedural bone rotation on RENAMED bones (§3b); baked clips are NOT the plan (no AnimationMixer for mobs). If the procedural walk looks stiff in the frame-sequence capture, bake a simple walk cycle in Blender from the rig (rotate thighs/calves/arms on keyframes, 1 s loop) — for the Woman, her baked walk clip is the bake source; for Worker/Zane, bake from the rig — doable, but budget the time. |
| 2 | **Zane Ryder is 109 MB.** Decimation + texture resize may still leave it over budget, or lose the face detail. **Per-human budget REVISED 2026-11-11 ~20:45 AST:** 0.9 MB GLB is unachievable (geometry alone at ~16–18k verts ≈ 0.9–1.1 MB; measured floors Worker 1.24 MB, Zane 1.82 MB). New budget 1.5 MB GLB per human. | Decimate in stages (50% → check → 25% → check). Textures to 1024 px, then 512 px if needed. Target ≤18k verts (see §7 vertex budgets). **Zane must fit 1.5 MB via 512 px textures + further decimation; if he can't, DROP HIM ENTIRELY** and use Worker/Woman only — using him for fewer mobs saves nothing (the GLB is embedded once regardless of how many mobs use it). The 32 MB hard cap stands: if the running total projects over 32 MB with all three humans, Zane is cut #3 (saves ~2.4 MB embedded) — decided at the running-total check, not pre-emptively. |
| 3 | **The Girl is "stylized" by name AND UNRIGGED (builder finding 2026-11-11 ~20:40 AST).** Both Girl FBXs (48k + 71k) are static meshes — NO armature, NO bones, NO actions. She cannot be a mob with the bone-rename adapter as-is. Kester picked her personally — her pick stands, and she is the ONLY female model (Kester wants random male AND female characters). | **RIGGED 2026-11-11 ~21:43 AST** (builder, `work/rig_girl.py` — manual 7-bone armature named directly to the mob slots, A-pose, 1,483,116 B, 16,515 verts, 1.69 m, all 6 slots verified by bone-swing pixel-diff). **COORDINATOR JUDGMENT 2026-11-11 ~21:53 AST (seen with my own eyes in the preview): she reads STYLIZED, not realistic — anime face with large blue eyes, blue-and-magenta two-tone hair, neck/chest tattoos, white crop top + short shorts.** Next to the realistic Worker/Zane she is a different art style. **DECISION: SHIP HER SPARINGLY** — female mobs at a lower spawn share (~1 in 4 mobs female), and tell Kester plainly she reads stylized. Do NOT drop her (she is the only female model; dropping her means all-male mobs). **KESTER'S DECISION 2026-11-11 ~20:00 AST: SHIP HER FOR NOW, swap her tomorrow** (once the model-selection audit lands and CGTrader downloads reset) for a better realistic female model. Do NOT hold step-4 waiting on a better model. **Swap-path audit (coordinator, 2026-11-11 ~20:05 AST — NO hard-coded references, swap is clean):** the Girl is wired only as the `mobGirlGLB` base64 const → `mobGr` loader slot → `mobGirl` model-map entry → `girl` role key in `useModels()`. Nothing references her height (1.69 m), materials, hair, tattoos, or bone names — the mob loop writes only the six generic adapter slots. Tomorrow's swap = replace the `mobGirlGLB` const (same `swap_const.py` pattern as the cow/chicken swaps) + preview-verify + in-game verify. Two conditions: (a) the new model exposes the same 6 adapter bone slots (rig it with the `work/rig_girl.py` pattern if unrigged, or rename bones); (b) it fits the 1.5 MB GLB budget. **One open check at swap time:** confirm the step-4 headshot-height re-derivation is per-body at runtime (from `M.h.head`), not hard-coded per role — if it is per-role, the swap needs that one code touch. **Model-selection audit guidance for tomorrow:** prefer an already-RIGGED female model, ≤ ~70k polys source, that can fit 1.5 MB after decimation (the current Girl sits at 1,483,116 B — right at the cap). **SUPERSEDED 2026-11-11 ~20:06 AST (Kester): "get everything tonight from Grab3D so there is NO swap tomorrow — use the right girl from the start." The "ship her sparingly, swap tomorrow" decision above is DEAD. The female mob is the Grab3D Woman Base Mesh (§2a(i): CC0, rigged, walk+run baked, 1.68 m, landed 2026-11-12 ~00:06 AST, conversion + walk gate in flight). The cartoon Girl does NOT go into the pool. She stays as the FALLBACK keeper (`preview/models/girl.glb`) and goes back in sparingly (~1 in 4) ONLY if the Woman fails the preview-gate walk test — and only on the coordinator's explicit instruction to the step-4 builder. The step-4 builder was told to hold the female slot as a clean swappable placeholder (Worker + Zane live, `mobGirlGLB` const uncommitted) until the Woman's keeper passes the gate. The swap-path audit above still applies verbatim to the Woman (same const/slot/role wiring, same two conditions: 6 adapter slots + ≤1.5 MB GLB). The one open check at swap time (headshot-height re-derivation per-body at runtime) still stands.** |
| 4 | **The Jeep may have no interior.** First-person driving needs one. | Verify on download. **Interior gate (§3b):** interior confirmed → first-person chase car; no interior → the Jeep is NOT drivable as a first-person vehicle — the fallback is a scoped chase-camera build (a new camera system, decided with Kester told plainly), NOT a silent "decide at build time". |
| 5 | **The Norton is 404k faces.** Decimation could mangle the chain/spokes. | Decimate the body aggressively, keep wheels/chain at higher density, verify in preview. If the chain won't survive, hide the chain (it's a detail, not a load-bearing part). |
| 6 | **Enterable interiors need furniture the downloads don't include.** | §4 furnishing plan: reuse village furniture if separable; else prop-library furnishing; flag "furnish pass 2" rather than blocking the build. |
| 7 | **The medieval block IS an unusable Lumion scene; the archviz scene is a 514 MB Blender scene.** | Medieval block DROPPED (no usable format, §4). Archviz is convert-or-drop (§4) — the builder inspects the `.blend`; if exterior meshes extract cleanly, use 1–2 as fillers; otherwise DROP. |
| 8 | **RESOLVED 2026-11-11 ~20:30 AST — L_build + Concrete House identities corrected.** `glb.rar` (the L_build GLBs) IS the New York Buildings (rickardosanchez, Royalty Free); `export_concrete_house_003.zip` IS the enterable Medieval House (Enterables, Royalty Free). | Identities corrected in §2a/§3b/§4; §9 item 0 is SATISFIED for all 23 landed files. No model ships with an unconfirmed origin. |
| 9 | **Single-file size.** 14.4 MB → ~32 MB target. Kester plays on a phone (DeX). | §7 budgets + the 32 MB hard cap. If the cap breaks, cut textures first, then models. |
| 10 | **The browser task is DONE (hit CGTrader's 25/day free-download limit). Only the dog is still expected (tomorrow, first priority).** | Build order (§9 step 12) lets ready swaps proceed; the dog slots in when it lands. If it never lands, the old dog (Cube) stays and the README says so — no silent gaps. |
| 11 | **The `house` (`p6`/`g.scene`) and `cottage` (`H6`/`B.scene`) GLB placements are unmapped.** | Builder traces their usage before touching them (§3c). The cottage GLB is the cottage wall/floor structure — it stays as-is in this build (see §4 known difference: cottages keep their position-zone shelter). Do not swap blind. |
| 13 | **Supply POI inventory is unverified.** §6 item 1 cites 5 of 11 POI positions from the bundle; the remaining 6 come from the build-start inventory of `world.pois`. | If the inventory finds a different count or values, §6 item 1 is restated from the inventory, not from the doc's examples. |
| 14 | **The Day 4 surge depends on the truck roadblock.** | If the truck swap (step 9) slips or the (0, -80) placement fails verification, the surge (§6 item 4) has no on-foot gate and must be re-scoped — it cannot ship as a drive-by. |
| 12 | **The chase stage is PROPOSED, not approved.** | No code. The design notes the dependency (vehicles) and stops there. |

---

## 11. Swap log (fill in as swaps land — also mirrored in README.md)

| Swap | Model | Source URL | License | Format in | GLB size | Pre-swap commit | Live verified |
|---|---|---|---|---|---|---|---|
| Hen → chicken (`q6`/`t.scene`) | Hen (free low-poly), author promiseijalade | CGTrader (per §2a inventory — page URL not in inventory) | Royalty Free | FBX → GLB | 230,052 B | f421b69 | YES — 2026-11-11 ~21:08 UTC: model-preview.html 200, live root hash == local db88457, hen.glb 200 (`.nojekyll` ab40008 fixed the deploy stall). Local gates all pass: preview bone-swing verified, post-embed byte-identical, headless in-game screenshot shows 3 hens at correct scale, skeleton clone 27/27 own bones, "Models loaded ✔" |
| Old Car 04 → wreck (`W6`/`c.scene`) | Generic Old Car 04, author Mavi3D | CGTrader (per §2a inventory — page URL not in inventory) | Royalty Free | FBX → GLB | 998,532 B | db88457 | YES — 2026-11-11 21:12:48 UTC: live root hash 419020c82dc16e020475d27ab3ba95754f15061a1f63ba00b8cadf3a2854a3b2 == local (poll try 2). Notes: 295k polys → 17,375 verts / 27,924 faces @ decimate 0.035; textures mostly missing (only LAMP/GAUGE JPGs) — flat materials; Rw() tints `body`/`chrome` per wreck, other materials keep originals; faces +Z like the old wreck |
| Cow → cow (`h6`/`M.scene`) | Cow (Holstein), author RetroStyleGames | CGTrader (per §2a inventory — page URL not in inventory) | Royalty Free | FBX → GLB | 331,548 B | 781e83e | YES — 2026-11-11 21:27:49 UTC: live root hash 435634308ef4ad944321d40e9d5dae55cc77b052951f78ae7511462b7b5492f3 == local (poll try 2). Notes: FBX armature +90° X — fixed by rotation-apply (cow_fixrot.py, bind pose preserved, verified by identical render+bbox); relinked T_Cow_B.png (2K→512 JPEG); hide/patch SPLIT by texture brightness (Holstein pattern survives, game tints work); bones legFL/FR/BL/BR + head (head = DEF-spine.018, vertex-weight-proven; spine.012–014 chain is dead rig); 1,909 verts / 2,246 faces; the 5 baked anim FBXs were NOT shipped (procedural gait); preview verified (proper Holstein); in-game probe: skeleton 66/66 own bones, legs=4, head=true, meshes=2 (hide/patch), screenshot shows black/white + brown/white Holsteins at correct scale |
| Bird → bird (`Z6`/`i.scene`) | Achara Bird, author pixol3d | CGTrader (per §2a inventory — page URL not in inventory) | Royalty Free | FBX → GLB | 291,028 B | a9619f4 | YES — 2026-11-11 21:37:35 UTC: live root hash 6eac7450ba843e543f2e66192dd19c1b5beaf299324f4bcf2806b5a22d494652 == local (poll try 2). Notes: FBX embeds 2K diffuse+normal textures (the 30 MB tex.rar was NOT needed); eye decimated 16,132→~160 verts, body 0.35, textures 128 px JPEG; 2,546 verts / 5,072 faces. **Armature-scale bake (work/bake_armature_scale.py):** the FBX left a 0.01 unit scale on the Armature node while POSITION data is metres — Box3.setFromObject (used by the game's cg() AND the preview viewer) double-counts it and reads the bird 100× small, so cg() would have scaled it 50×. Bake removes the Armature scale, scales bone translations + animation translation keyframes ×0.01, AND rescales the inverse-bind matrices (IBM_new = S × IBM_old, first 3 rows ×0.01 — translation-only is WRONG, it leaves the render 100×). Verified numerically: max \|boneWorld × IBM − I\| = 6e-7 over all 59 joints; viewer then reads 1.38×0.70×1.40 m and renders the bird correctly. Forward = +Z (head z=+0.55, tail z=−0.38) — matches the game's rotation.y=−a flight math (local +Z maps to velocity). The game's wl/wr procedural wing-flap is a no-op after the swap (it targets the removed primitive wings); the baked clip does the flapping. In-game probe: 9 birds, 9 mixers, clip `Armature\|ACHARA ENSAMBLE_TempMotion` playing (time advancing), skinned, 4 meshes; screenshot shows a bird in flight at ~20 m |

---

## Appendix A — PROPOSED chase stage (DESIGN ONLY — NOT APPROVED, DO NOT BUILD)

**Status:** concept draft, 2026-11-11 ~15:50 AST; settled further 2026-11-11 ~15:52 AST
(see A5). The concept is now essentially settled — **but still NOT approved for build.
No code for this stage until Kester explicitly approves the full concept.** This appendix
exists so the idea is captured, not so anyone builds it.

### A1. Kester's flow
1. Complete a stage mission → Kes finds the Jeep Renegade (door knob glows).
2. Mission options appear (e.g. **"Evade road pirates"**).
3. Cannot proceed without entering the vehicle (gated) — door knob glows per his rule,
   prompt **"Get in"**.
4. Once inside, the mission starts.
5. Mad Max villains attack from everywhere as he drives.
6. He can shoot while driving.
7. First FORWARD shot shatters the windscreen — then he can shoot freely forward.

### A1b. Why they chase him (Kester, settled 2026-11-11)
"The game is apocalyptic survival choices… in real life you'll be attacked for your stuff."
The road pirates chase Kes BECAUSE he has stuff worth taking — his supplies (food, water,
fuel) in the car. The chase's stake is keeping the stuff: **win = reach the village gate
WITH the supplies; lose the car / get caught = lose the stuff, not just health.** The
supplies in the car are the reason the chase exists at all — no supplies, no pirates.
This is the standing principle at the top of this doc applied to the chase: carrying more
makes you a bigger target.

### A2. Logic improvements (parent's draft, presented to Kester)
1. **The gate is the door-knob rule.** The car door knob glows on approach (his own rule)
   so the player knows to get in. The world waits — the mission does NOT start until he
   is inside.
2. **Villains don't pop in from nowhere.** A lookout spots him and radios ahead; they come
   in escalating waves from logical places (side roads, gas station, behind wrecks, over a
   hill): wave 1 = two bikes; wave 2 = a ramming car; wave 3 = a truck blocker ahead + bikes.
3. **Shooting while driving costs steering.** One hand off the wheel — steering gets loose
   while shooting. Aim assists to the nearest threat (arcade, not fiddly).
4. **Windscreen logic fixed.** SIDE shots work from the start (side window open / lean out —
   machete for alongside bikers, shotgun for cars at distance). Only FORWARD shots are
   blocked by the windscreen, and the first forward shot shatters it (glass moment, brief
   blur, then clear). Breaking it has a cost: they can now throw/shoot things in through
   the open windscreen too. Their attacks can also crack it first.
5. **Ammo is limited.** Shotgun shells — few, every shot counts.
6. **Win = reach the village gate with the car still running AND the supplies intact.**
   Two meters: vehicle condition + Kes health — and the supplies themselves are the third
   thing at stake (see A1b): lose the car / get caught = lose the stuff, not just health.
   Running off the road = crash damage per the existing fall/crash rules.
7. **A Choices fork mid-chase:** highway (fast, open, more pursuers) vs back roads
   (tight, wrecks, fewer pursuers).
8. **Assets:** the military truck serves as the villain war-rig blocker; the mob is the
   same angry mob from stage 1, now motorized (bikes = Norton Manx model reused for
   villains; ramming car = the sedan — the pirates' beater. The Jeep is the car Kes finds
   and drives, NOT a villain car. No car belongs to anyone — the villains drive what they
   find too).

### A3. Awaiting Kester's call — UPDATED 2026-11-11 ~15:52 AST
- ~~Which car Kes spawns next to~~ — CORRECTED by Kester 2026-11-11 (see A5 item 3): the
  Jeep Renegade — the car Kes FINDS and drives in the chase. The old car is OUT ("nah swap
  that shitty car lol"). No car belongs to anyone; the Jeep is found, not owned.
- ~~Whether a go-on-foot alternative exists~~ — SETTLED by Kester (see A5 item 1): no, the
  chase is the only way forward for that stretch.
- ~~Ammo count~~ — SETTLED by recommendation (see A5 item 3): 6 shells, tunable. Kester has
  not overruled it.
- **What remains: Kester's explicit approval of the full concept.** Until he gives it, no
  chase-stage code — this appendix stays design-only.

### A5. Settled design (2026-11-11 ~15:52 AST)
1. **The chase is the ONLY way forward for that stretch (Kester).** No go-on-foot
   alternative until the checkpoint. The gate (must enter the vehicle) is absolute for
   this stretch of the game.
2. **The ending is BOTH, working together (parent's call).**
   - **FUEL is the chase's clock / pressure.** Limited fuel; boosting burns it faster. You
     cannot dawdle or drive in circles — every second and every boost costs fuel.
   - **The CHECKPOINT is the exit.** Reach it and you can get out of the vehicle on your
     own terms and continue on foot into the next stage.
   - **If fuel runs out BEFORE the checkpoint, the car dies where it is.** You are stranded
     on foot in hostile ground, and you can only carry what fits in your pack — the rest
     of your stuff stays in the dead car (and can be looted). **Running dry costs supplies,
     not just time.** The fuel gauge and the checkpoint distance are the two numbers the
     player watches the whole chase.
3. **Recommendations for the last two open bits (parent's; Kester has not overruled them) —
   UPDATED 2026-11-11: the car question is no longer a recommendation, it is Kester's call.**
   - Kes finds the **JEEP RENEGADE** — the car he drives in the chase (Kester, 2026-11-11:
     "nah swap that shitty car lol" — the old car is OUT as his ride). The Jeep belongs to
     no one; he finds it and drives it. The truck / bike are for villains and variety; the
     old car is a roadside wreck.
   - The shotgun carries **6 shells** for the stage as the starting number (tunable).
4. **The settled flow, end to end.** Mission complete → Kes finds the Jeep Renegade (door
   knob glows) → mission options ("Evade road pirates") → must enter the vehicle to start (the
   chase is the only way forward) → lookout spots you, waves escalate from logical places
   (bikes sideswipe, cars ram, truck blocks) → shoot while driving (loose steering
   trade-off; side shots free, first forward shot shatters the windscreen, breaking it
   lets them throw things in too; limited ammo, 6 shells) → fork choice mid-chase
   (highway vs back roads) → reach the checkpoint with fuel left (clean exit on foot into
   the next stage) or run dry (stranded on foot, lose supplies) → next stage. The reason
   for the chase: they want his stuff (supplies in the car).

### A4. What the build does about this stage today
Nothing. The vehicle swaps in §3b (Norton rideable, Jeep — the car Kes finds and drives,
truck roadblock) are the only chase-enabling work in this build, and they are justified on
their own (free-roam driving + the whole-game upgrade, Stage 1 included) even if the chase
stage is never approved. If Kester approves the concept later, a separate design pass specs
the stage (spawn logic, wave director, vehicle condition meter, windscreen state machine,
fork choice) before any code.

---

*End of design doc v1. Goldfish review next.*
