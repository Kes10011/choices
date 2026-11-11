# Choices — Asset Integration Design Doc

**Status:** DRAFT v1 (Elephant) — 2026-11-11 ~15:50 AST; revised 2026-11-11 ~20:00 AST
(Goldfish 3/3 resolved) and ~20:20 AST (Goldfish 1/3 + 2/3 resolved). **ALL THREE GOLDFISH
REVIEWS RESOLVED — the doc is now vetted and buildable.**
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
- The archviz scene (491 MB Blender render file) — too heavy, render-only. Deferred.
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
**Licenses: verify per model from the browser task's final report when it lands.**
CGTrader free models carry VARYING licenses (some editorial / attribution / non-commercial)
— Royalty-Free is NOT the default; verify per model and record the exact license per model
in the README swap log (§11). The builder does NOT ship any model whose license is unclear
or non-commercial. **No browser-task final report exists in the workspace yet** — until it
lands (or a page re-check is filed), the 8 landed models have NO recorded source URL or
license; §9 item 0 blocks their commits until that document exists.

### 2a. Landed (11 files — re-frozen 2026-11-11 ~19:57 AST; files 1–8 extracted, files 9–11
landed, extraction pending)

| # | Archive (as landed) | Model | Contents (verified) | Size |
|---|---|---|---|---|
| 1 | `worker_caucasian_fbx.rar` | **Worker Character** (male) | `worker fbx.fbx` (2.6 MB) + 7 textures (worksuit, skin, eyes, teeth, tongue, shoes). Full humanoid rig: calf/foot/hand/head/lowerarm/spine_01-03. **Animation status UNVERIFIED until Blender inspection — a `strings` grep cannot prove a clip is absent (a `Takes` node IS present in the file).** | 19.6 MB |
| 2 | `FBX.7z` | **Wild Stylized Girl** (female) | `Girl_LOW_48092Vertex.fbx` (3.0 MB) + `Girl_LOW_71070_Vertex.fbx` (4.4 MB) + Texture/ (BaseColor, Normal, Eye, Accessories). Rig/animation status: NOT yet verified — check in Blender. Use the 48k version unless the 71k version is visibly better in a side-by-side preview (reads more realistic next to the Worker, cleaner face/hands) — if so, use the 71k version. | 93 MB |
| 3 | `Zane_Ryder_fbx.zip` | **Zane Ryder** (extra mob) | Single `Zane Ryder fbx.Fbx` (109 MB). CC_Base rig (BoneRoot, Head, JawRoot, Spine01/02). **109 MB is far too big to embed — decimate + resize textures hard (see §3).** | 106 MB |
| 4 | `Hen.rar` | **Hen** | `hen.Fbx` (418 KB) + `hen.blend` + `hen.json` + textures (mati_chicken PBR: roughness/metallic maps only + flat diffuse color [191, 127, 0] — NO diffuse texture). Bip01 rig (Head, L/R Thigh, L/R Foot, Spine). Single mesh `skel_chicken`, material `mati_chicken`. | 1.0 MB |
| 5 | `Gledista_Triacanthos_FBX.zip` | **Honey Locust tree** | 6 FBX variants + maps/ (bark, beans). 169 MB total — pick 1–2 variants by eye, do NOT embed all 6. | 161 MB |
| 6 | `MT_PM_V060_FBX.7z` | **Maxtree Plant Models Vol 60** | 15 FBX files. **Extracted ONLY the 6 grasses** (17 MB total): Cynodon_dactylon (5.8 MB), Cyperus_eragrostis (0.8 MB), Digitaria_sanguinalis (1.6 MB), Lolium_perenne (4.6 MB), Setaria_viridis (0.8 MB), Zoysia_japonica (4.2 MB). The 9 trees are 27–299 MB each — NOT extracted, NOT used. | 727 MB |
| 7 | `glb.rar` | **L_build_1/2/3** (unidentified origin) | 3 GLBs (~100–138 KB each). Each: 3 nodes (LOD0/1/2), 3 meshes, **NO materials, NO animations**. Three building shells with LODs. Origin MUST be confirmed from the browser report before shipping. | 59 KB |
| 8 | `export_concrete_house_003.zip` | **Concrete House 003** (unidentified origin) | LOD0/1/2 in 3ds/fbx/obj + textures (concrete, roof, redwood). No interior evidence. Origin MUST be confirmed from the browser report before shipping. | 90 MB |
| 9 | `browser-download-20261111T193435.221611111Z-8--t_hut_007_export.zip` | **t_hut_007 — medieval hut/cabin** | `export/model/`: LOD0 + LOD1 in 3ds/fbx/obj (+ .mtl). Timber hut — this IS the medieval house/cabin. Interior state NOT yet verified — check in Blender (door opening? interior faces? wall meshes separable?) before §9 step 8. | 108 MB |
| 10 | `browser-download-20261111T193810.994165533Z-9-Medieval_Block_9.ls11` | **Medieval Block 9** | Single `.ls11` file — Lumion scene format (binary; `file` reports "data"). NOT convertible by Blender. **SKIP for this build** per §4 — do not spend build time on it. | 663 MB |
| 11 | `browser-download-20261111T195637.206524133Z-10-small_village_fbx.zip` | **Small village** | Single `small village.fbx` (71.5 MB). 4 homes per the page (2 furnished, 1 empty, 1 closed) — verify the count + interiors + separable furniture in Blender before §9 step 8. | 65 MB |

### 2b. Still expected + per-swap dependency rule (Goldfish fix — replaces the old blanket ban)
**Rule:** there is NO blanket "do not start until everything lands". Each swap depends on
specific files; a swap whose file hasn't landed is SKIPPED with a clearly-marked README
TODO, and all other swaps proceed in §9 order. The build starts when Blender is verified
(§9 step 1), not when the last download lands. The per-swap table below is re-checked at
build start and the "Landed?" column updated.
**Still expected (browser task still running):** cow · small bird · dog · New York buildings ·
windmill · archviz scene · Remington Model 31 shotgun · machete · yellow flashlight ·
Norton Manx motorcycle · Jeep Renegade SUV · Generic Old Car 04 · Military Truck.
**Landed since the last freeze:** medieval hut/cabin (t_hut_007) · small village · medieval
block (.ls11 Lumion — skipped, see row 10 above).

| Swap (§9 step) | Required file | Landed? (2026-11-11 ~19:57 AST) |
|---|---|---|
| Hen → chicken (step 2) | Hen.rar | ✅ YES |
| Old Car 04 → wreck (step 3) | Generic Old Car 04 | ❌ NO — skip with TODO |
| Mobs + Stranger (steps 4–5) | Worker / Girl / Zane | ✅ YES |
| Weapons → viewmodels (step 6) | Shotgun / machete / flashlight | ❌ NO — skip with TODO |
| Trees + grass (step 7) | Honey Locust / Maxtree grasses | ✅ YES |
| Enterable buildings (step 8) | t_hut_007 ✅ · small village ✅ · windmill ❌ NO (windmill swap waits) | PARTIAL |
| Vehicles (step 9) | Jeep / Norton / Military Truck / Old Car 04 | ❌ NO — skip with TODO |
| Fillers (step 10) | NY buildings ❌ · L_build ✅ (origin TBD) · Concrete House ✅ (origin TBD) | PARTIAL |
| Cow / dog / bird (slot into steps 2/7) | Cow / dog / small bird | ❌ NO — skip with TODO |

### 2c. Vehicles — page facts + decisions
| Vehicle | Page facts (checked on CGTrader) | Decision |
|---|---|---|
| **1947 Norton Manx** (motorcycle) | Blender 2.7, 119 objects, **404,291 faces**, UV textures, OBJ/FBX/DAE/3DS. Chain has individual pieces (rigging possible — wheels can spin). Minor clipping on piping/wiring. | **RIDEABLE — Kes rides it.** Same driving bar as the car (Far Cry 2 feel, proper handling, no reversed steering). Decimate to ≤50k faces. Arcade handling matched to the car, lean visual on turns, no tip-over death. Rider pose: seated, hands on bars. |
| **2016 Jeep Renegade SUV** | Low-poly, full-scale, real photo-textures, "very light and fast to render", simplified modelling. Author: José Bronze. | **THE CAR KES FINDS AND DRIVES IN THE CHASE — primary drivable + chase vehicle (Kester, 2026-11-11: "nah swap that shitty car lol").** The Jeep replaces the sedan as the chase car. Kes finds it after a stage mission (chase gate) and drives it in the chase. No car belongs to anyone — it is found, not owned. Handling: heavier, top speed ~10, more stable. OPEN: interior for first-person driving view NOT confirmed on the page — verify on download; if no interior, chase cam for the Jeep only (the sedan interior is part of the sedan GLB and cannot be reused). |
| **Generic Old Car 04** | High-quality exterior + **medium detailed interior**, real dimensions, textures included, FBX/OBJ, objects separated and named. Author: Mavi3D. | **WRECK / SCENE PROP ONLY — demoted (Kester, 2026-11-11).** A broken-down roadside car fits the apocalypse. It is NOT drivable and NOT the chase car — it is a shitty car nobody would choose to drive (not because it belonged to anyone: no car belongs to anyone). Replaces the current wreck (`W6`/`c.scene`) and appears as roadside wrecks. The Wounded Stranger sits against a wrecked sedan — this is the natural model for that scene. |
| **Military Truck** | Page facts UNVERIFIED — page fetch failed 2026-11-11 15:43 (empty response). Verify on download. | **SCENE / WRECK VEHICLE.** Military convoy wreck as a landmark + cover + mission site (search the truck bed for supplies). Can be promoted to drivable in a later pass if it has an interior and reasonable poly count — NOT in this build. |

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
| `h6` → `M.scene` | cow (357 KB) | **Cow (PENDING download)** | Same bone-rename pattern as the hen: rename the leg bones to `legFL/legFR/legBL/legBR` and the head bone to `head` (**NO Empty proxies — they are inert on skinned meshes**). Materials `"hide"`/`"patch"` are material renames (fine). Do NOT swap until the cow file lands and is inspected. If the cow model has no leg bones to rename, the swap is blocked until a rigged cow is found or the code path changes. Scale: `cg(M.scene, 2.6)` is a footprint, not a height. |
| `d6` → `Q.scene` | dog (116 KB, **single Cube mesh** — the dog IS a cube) | **Dog (PENDING download)** | The old "dog" is one Cube; the swap replaces it with a real dog GLB in the same slot. Dog/cat use the shader rig (`rig()` — bounding-box leg swing, NO node names needed). **Candidate dog GLBs must pass the shader-shear check in §8** — the shader swings vertices below 55% height, and a skinned/rigged GLB dog may double-animate or shear under it; "any quadruped-shaped dog GLB works" is overconfident. Verify the shape reads as a dog at 0.95 footprint (`cg(Q.scene, .95)`) and the leg-swing shader doesn't shear the head. Fallback if it fails: static pose (no leg swing) — do not ship a sheared dog. |
| `Z6` → `i.scene` | bird (185 KB, `KeyAction` ×3 anims) | **Small bird (PENDING download)** | Game plays `birdClips[0]` if present, else static. If the new bird has a baked flap clip, keep it as clip 0. If it has no animation, the builder adds a code wing-flap (rotate wing nodes) — do NOT ship a frozen bird. (The old "relies on the primitive birds' wing planes" fallback is wrong: the current bird GLB has `KeyAction` anims, not wing planes.) |
| `W6` → `c.scene` | wreck / car #2 (87 KB) | **Generic Old Car 04** | Direct swap. Verify scale vs the current wreck (measure bounding box) and orientation (front faces the same way — the stranger sits against its side at (2.2, 7.2)). Interior exists (medium detail) — good for the stranger scene. |
| `_6` → `r.scene` | oak tree (43 KB) | **Honey Locust** (1 of the 6 variants) | Pick the best-looking variant by eye in the preview. **The `G` tree array CANNOT be extended to a 4th variant this way** — `G` is built by cloning nodes named `leaves`/`tree` with suffixes `''`, `.001`, `.002` from the TREES GLB; the Honey Locust is a separate GLB with different node names and no `.003` variant. A second Locust variant needs its OWN loader path (own node names, measured-height scaling) — write it separately and leave the `G` suffix loop untouched. The oak uses `r.scene` RAW (no `cg()`), so the Locust must match the oak's native scale, not a `cg()`-normalized one. **One embedded const, referenced twice — the oak replacement and the 4th tree variant share the SAME base64 const; do NOT duplicate the bytes.** |

### 3b. Additions (new consts / new code paths — no old slot removed)

| New object | Source | Integration |
|---|---|---|
| **Mob humanoids** (replaces `LS()` primitive builder) | Worker Character (male) · Wild Stylized Girl (female) · Zane Ryder (extra) | New loader entries + a mob-model pool. Mob groups get a MIX of the three bodies (random male/female, no names). **Adapter — BONE RENAMES, one scheme, decided now.** The mob update loop writes SIX slots unguarded every frame (`legL/legR` gait, `armL` counter-swing, `armR` windup −2.5 / recover, `torso` rotation.z/x, `head` rotation.x=.18 on chase) — no optional chaining, no fallback. A mob GLB missing ANY slot throws a `TypeError` every frame or leaves the mob frozen. **The doc chooses scheme (i): the Blender adapter maps real bones to the exact slots the loop expects.** Rename the bones in Blender per body: thigh groups → `legL`/`legR`, upper-arm groups → `armL`/`armR`, `spine_02` → `torso`, head bone → `head`. (Bone renaming is safe — skinning is index-based.) NO Empty proxies for animation — an Empty parented to a bone is inert on a skinned mesh. **Empties ARE still fine as static attach points** (they follow the bone they are parented to): the weapon socket is an Empty parented to the `armR`/hand bone — the primitive builder parents the weapon to an arm child, and the new hands need the same attach point. **Headshot heights:** `ray()` uses hardcoded heights 1.64/1.1 m, not the head node — re-derive them from the model (or from `M.h.head` world position) per body, or headshots misalign on taller/shorter mobs. **Animation:** the builder drives the walk procedurally (rotate the renamed bones with `sin(phase)` in the mob update loop, mirroring the existing `gait()` pattern; swing arms opposite; lean `torso` forward slightly on chase). **Baked walk clips are NOT the plan** — the code has NO AnimationMixer for mobs (mixers exist only for birds/fire), so "prefer a baked walk clip" is unimplementable without adding a mixer (a scoped code change the builder does NOT make unless the coordinator approves it). If a baked walk clip IS found in any of the three, note it; the procedural walk is the default. Verify in Blender before deciding, and verify in the preview that rotating each renamed bone swings on the expected axis. **Walk gate (§8/§9):** the mob GLB must expose all six animation slots + the weapon socket, verified by GLB JSON + a headless walk-cycle frame-sequence capture — the coordinator judges it; stiff = reject, bake a walk cycle in Blender (§10 item 1). Mob HP (55), speed, weapon (cleaver/bat/pipe/crowbar — keep the existing weapon meshes for now; the new models' hands hold them via the weapon socket) and spawn logic are UNCHANGED. **Mob weapons stay primitive in this build** (no CGTrader mob-weapon models were ordered) — see the residual-boxy list in §3c. |
| **Wounded Stranger** (replaces `P6()` primitive sitter) | Worker Character, seated pose | High priority — this is the Day 2 emotional peak. Pose: seated, back against the sedan, one leg bent with the rag/blood detail. Implement the pose in Blender on a SEPARATE COPY of the Worker file: rotate spine/hips/thighs, then Pose → Apply Pose as Rest Pose (glTF exports the rest pose + animations — a Pose-Mode pose does NOT survive export otherwise, and applying it as the rest pose destroys the rig for reuse, hence the copy). Keep the breathing animation — drive it through the renamed `torso` bone — BUT retune the baseline: the current `oP.rotation.x=.25` was tuned for the primitive torso and will permanently hunch the seated figure on a `spine_02` bone. Retune in the preview. |
| **Shotgun viewmodel + world pickup** | Remington Model 31 (PENDING) | Replaces the primitive gun viewmodel (`vms.gun`). **Needs a muzzle anchor node** (the muzzle-flash effect attaches to it) — add an Empty named `muzzle` at the barrel tip in Blender (an Empty is fine here: it is a static anchor for the flash effect, not a skinned bone). **Viewmodel integration spec:** viewmodels are built SYNCHRONOUSLY by primitive builders (`bS`/`OS`) returning `{g, muzzle}`; the gun's rest position/rotation are hardcoded in `vmBase` (gun at (.22, −.2, −.32)) and the muzzle-flash class consumes `i.muzzle`. The builder PRELOADS the shotgun GLB at startup and the viewmodel builder CLONES the loaded scene (it cannot parse a GLB synchronously). Per-weapon scale + `vmBase` retuning verified by first-person screenshots. **World pickup: DELETED as specified** — the player already owns all three weapons (keys 1/2/3, ammo 18 at start), so a pickup granting a weapon they already have does nothing, and there is no pickup code path for it (`nearestSupply` handles only POI and ammo; caches are re-randomized every stage by `spawnAmmo()`, so there is no fixed "near" to put anything). If a shotgun pickup is wanted, it must grant something real (e.g. +6 shells) with a defined `nearestSupply` kind, radius, and fixed coordinates — decided in §6 item 1, not here. Keep the existing gun stats (ammo, damage, shotCd) — this is a visual swap, not a balance change. |
| **Machete viewmodel + world pickup** | Machete (PENDING) | Replaces the primitive machete viewmodel (`vms.machete`). Same loader pattern as the shotgun (preload at startup, clone into the synchronous builder; `vmBase` retuning per weapon). The swing animation is code-driven (viewmodel rotation) — the new model must be oriented so the blade points the same way as the old one (verify in the preview against the old viewmodel's rest pose). World pickup: same as the shotgun — deleted as specified (the player already owns the machete). |
| **Flashlight viewmodel** | Yellow flashlight (PENDING) | Replaces the primitive light viewmodel (`vms.light`). Same loader pattern as the shotgun (preload, clone, `vmBase` retuning). The actual light cone (`this.flashlight` spotlight) is unchanged — only the in-hand model swaps. Verify the model has a lens/front face the light can plausibly come from; if the model is yellow plastic, keep the yellow — it reads on camera. |
| **Norton Manx** (rideable) | 1947 Norton Manx | NEW vehicle system. Reuse the car driving code path with a bike variant: same W/S/A/D/SPACE/E controls, same no-reversed-steering rule, arcade handling, top speed ~12 (between car 11.5–13). **Camera decision (settled): first-person, handlebars + front wheel visible, NO visible rider body** — the same rule as the car (interior + wheel, no body). A visible seated rider and a first-person view cannot both be true; the first-person view wins. **Lean = camera roll in first-person** (the car code already rolls the camera: `rotation.z = -s*R*.004`) — specify lean as a camera-roll value, not a model roll. First-person anchors (handlebar node name, camera height/offset) are specified at build time from the model — the bike has no steering-wheel node, no cockpit, no seat height, so the builder defines them. **Per-vehicle parameters (new — the driving code has one global clamp, no per-car fields):** top speed, accel, steering factor, seat height + anchor node, camera mode — added to the car object per vehicle. Wheels: the chain/wheels have individual pieces — verify the front/rear wheel nodes are separable so they can spin with speed. **Decimate 404k faces → ≤50k.** **Spike-test:** the Norton is one of the riskiest conversions (119 objects, 404k faces) — spike-test it FIRST (§9 step 1), before the Hen proves the pipeline. **Settled-chase constraint:** the bike must work as a pirate vehicle (Appendix A). |
| **Jeep Renegade** (the car Kes finds and drives) | 2016 Jeep Renegade | NEW entry in `world.cars` (the car system already supports a list — `nearCar()` iterates it). **THE CAR KES FINDS AND DRIVES IN THE CHASE — primary drivable + chase vehicle (Kester, 2026-11-11: "nah swap that shitty car lol").** Kes finds it after a stage mission (chase gate) and drives it in the chase. No car belongs to anyone — it is found, not owned. **Per-vehicle parameters (new):** top speed ~10, accel, steering factor (heavier, more stable = less slide), seat height + anchor node, camera mode — added to the car object (the driving code has one global clamp, no per-car fields). **Camera decision (settled):** first-person IF the Jeep has an interior (verify on download). **If it has NO interior, the Jeep is NOT drivable in this build as a first-person vehicle — the fallback is a scoped chase-camera build** (third-person camera behind + above the vehicle, same W/S/A/D controls), explicitly scoped as a new camera system, NOT a silent "decide at build time". The builder does NOT silently ship a chaseless Jeep, and does NOT "reuse the sedan interior framing" (the sedan interior is part of the sedan GLB and cannot be reused). **Interior gate:** the Jeep's drivability is gated on the download inspection — interior confirmed → first-person chase car; no interior → chase-cam scope decision, with Kester told plainly. Add a map POI for it once discovered (see §6 item 6). The three existing cars (pickup, sedan, station wagon) stay drivable in free roam. **Settled-chase constraint:** the Jeep must work as the chase car (Appendix A). |
| **Military Truck** (scene/wreck) | Military Truck (PENDING) | Place as a roadblock at (0, -80), angled across the main road, with collision — see §4 placements. It blocks the road for VEHICLES (full-width; cars cannot pass) with a ≥1.5 m pedestrian gap on the west side, so the last ~20 m to the checkpoint is always on foot. Mission site: searching the truck bed gives a bandage pickup — reach-from-ground radius 2 m at the tailgate (the bed is ~1.5 m up; no climb mechanic exists; the pickup sits at the tailgate, outside the truck's collision footprint — `nearestSupply` measures 2D distance). NOT drivable in this build. **Settled-chase constraint:** the truck must be promotable to a villain war-rig (Appendix A) — keep its GLB intact for that. |
| **Medieval house** (enterable) | Medieval house/cabin (PENDING) | See §4. |
| **Windmill** (enterable) | Windmill (PENDING) | See §4. |
| **Small village** (3 enterable homes) | Small village (PENDING) | See §4. |
| **New York buildings** (filler) | New York buildings (PENDING) | Skyline/backdrop at the map edges (beyond the playable clamp, x ±112 / z -112..114). No collision needed beyond the map bounds. No interiors. |
| **L_build_1/2/3** (filler) | glb.rar (origin TBD) | Street filler buildings inside the villages. **They have NO materials** — the game's retexturing (`Rw()`) keys off material names, so materials must be AUTHORED in Blender (assign brick/plaster materials with names the game can key off), not "applied from a set". Check in Blender for vertex colours first. Use LOD0 in-game (LOD1/2 are for distance — the game doesn't do LOD switching, so ship LOD0 only). Do NOT ship until origin/license is confirmed from the browser report. |
| **Concrete House 003** (filler) | export_concrete_house_003.zip (origin TBD) | Modern house filler (LOD0 — LOD0 is the highest detail). Do NOT make it enterable in this build (no interior data). Do NOT ship until origin/license is confirmed. |
| **Maxtree grasses** (terrain) | 6 grass FBXs | Replace the instanced grass-tuft geometry: the game draws 4400 instanced tufts from ONE `InstancedMesh` with ONE geometry (0.9×0.7 plane) and ONE material (alphaTest .35, DoubleSide). **One InstancedMesh cannot mix species "randomly per instance".** Each Maxtree species is a multi-mesh, multi-material plant — it must be MERGED/BAKED into one geometry + one material PER SPECIES (merge step in §7), then one InstancedMesh per species (2 species: Cynodon + Zoysia — the smallest, most grass-like), per-plant budget ≤2k polys, instance count ~1500–2500 total (full 3D plants cost more than crossed planes; ~2,000 × ~2k polys ≈ 4M polys needs the §9 fps gate). Material: alphaTest .35 handling per species (Maxtree grasses are typically alpha-tested leaf cards). Keep the same placement rules (the `gA`/`iA` exclusion zones around buildings/roads). |
| **Honey Locust** (tree variety) | 1–2 of 6 variants | See the oak row in §3a: one Locust variant replaces the oak const (`_6`/`r.scene`); a second variant needs its OWN loader path (the `G` suffix loop cannot be extended). Do not embed more than 2 variants. |

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
- `f6`/`U` barrel, `X6`/`E` crate, `$6`/`n` pine, `AS`/`b` dead tree, `PS`/`y` far tree,
  `iS`/`T` shed, `TS`/`s` fire (campfire with animations — already works).

**Residual boxy list (known + accepted in this build — the doc does NOT claim the swap fixes
"boxy" for these):** mob weapons (cleaver/bat/pipe/crowbar stay primitive — no CGTrader
mob-weapon models were ordered); cat, barrel, crate, shed, house, cottage, pine/dead/far
trees stay as-is. Kester is told plainly: these are the accepted remainder, not an oversight.

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
| Medieval house | ✅ YES (page: "fully ENTERABLE") | EMPTY — "waiting for you to add your props" | Cut doorway (verify the source model has a door opening; if the wall is solid, cut it in Blender). Furnish: table + bed + crates. Furniture source: first check whether the small-village file has separable furniture meshes — if yes, reuse them; if no, furnish from the in-game prop library (crate/barrel) + a simple table/bed built from the house's own plank textures. Flag as "furnish pass 2" if the first pass is thin. Place at (-30, -44) — a mission interior for Day 2/3 (near the Alley Fire Site / The Field Lab). |
| Windmill | ✅ YES (page: "fully ENTERABLE") | EMPTY | Same as the house, lighter furnishing (sacks, barrels). Single floor only — NO ladder to an upper floor in this build (see the placements table). Place at (30, -60) as a landmark + mission interior on the village edge. |
| Small village (4 homes) | ✅ 3 of 4 (2 furnished, 1 empty, 1 CLOSED) | 2 furnished (ready), 1 empty (furnish lightly), 1 closed (stays shut) | Place as a distinct second village cluster, anchor (-35, -70), homes spread ±15 m, ≥15 m off the road centreline (see the placements table). The 2 furnished homes are ready-made mission interiors. The Day 3 shelter decision ("Where to Sleep") marker stays where it is — the village homes grant shelter buffs through their own `inShelter()` rects but do NOT move the decision (the decision triggers at the fixed Cellar Hatch marker (17.8, -38.5, radius 3.4); adding shelter rects elsewhere doesn't "wire" anything to it). The cluster is ~60 m+ from the hatch, in the dark, on Night 3 — the homes are usable shelter that night, the decision itself stays at the hatch. The CLOSED home is the fourth home, north-east in the cluster, boarded door: no glow, no entry, solid collider. E on it gives a "Locked — boarded shut" toast (a player who presses E and gets nothing reads it as broken, not deliberate). Its knob is present but NEVER emissive. |
| New York buildings | ❌ filler only | — | Skyline/backdrop. No doorway, no interior, no glow. |
| Medieval block (Lumion scene) | ❌ filler only (probably unusable — Lumion format) | — | If the download is a Lumion `.ls` scene, SKIP it for this build. Do not spend build time converting render scenes. |
| Archviz scene | ❌ deferred | — | 491 MB Blender render file. Not for this build. |
| Concrete House 003 | ❌ filler only (this build) | No interior data | Exterior filler. Revisit enterability only if a later inspection finds an interior. |
| L_build_1/2/3 | ❌ filler only | No interior data | Street filler. No glow. |
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
| Each human (Worker / Girl / Zane) | ≤ 0.9 MB | ≤ 1.2 MB | Zane is 109 MB source — decimate + 1024 px textures. If a human can't fit 0.9 MB, drop texture to 512 px before cutting geometry. If Zane still won't fit, DROP HIM ENTIRELY — using him for fewer mobs saves nothing (the GLB is embedded once regardless of how many mobs use it). |
| Hen | ≤ 400 KB | ≤ 0.53 MB | Source is 418 KB FBX — should fit easily. |
| Cow / Dog / Bird (when they land) | ≤ 600 / 600 / 300 KB | ≤ 0.8 / 0.8 / 0.4 MB | |
| Honey Locust (1 variant) | ≤ 600 KB | ≤ 0.8 MB | One const, referenced twice (oak + 4th tree variant) — counted ONCE. |
| Each grass species (2 max) | ≤ 250 KB | ≤ 0.33 MB | 2 species max (was 3). |
| Medieval house | ≤ 1.0 MB | ≤ 1.33 MB | |
| Windmill | ≤ 800 KB | ≤ 1.07 MB | |
| Small village (4 homes) | ≤ 1.0 MB | ≤ 1.33 MB | If the village ships as 4 separate GLBs, the 1.0 MB budget is the TOTAL for all four. |
| Norton Manx | ≤ 1.0 MB | ≤ 1.33 MB | 404k faces → ≤50k faces first. |
| Jeep Renegade | ≤ 1.0 MB | ≤ 1.33 MB | Low-poly source — should fit. |
| Old Car 04 | ≤ 1.0 MB | ≤ 1.33 MB | |
| Military Truck | ≤ 1.0 MB | ≤ 1.33 MB | |
| Shotgun / Machete / Flashlight | ≤ 200 KB each | ≤ 0.27 MB each | |
| **CORE TOTAL added (GLB)** | **≤ ~13.1 MB** | **≤ ~17.5 MB embedded** | |
| Less: removed GLBs (chicken 154 + cow 357 + dog 116 + bird 185 + wreck 87 + oak 43 KB ≈ 0.92 MB GLB ≈ 1.23 MB base64 freed) | | **−1.23 MB** | |
| **NET added to `index.html`** | | **≤ ~16.2 MB** | Current file 14.4 MB → target total **≤ ~30.6 MB**, hard cap **32 MB**. |
| Fillers — ONLY if budget remains: NY skyline ≤ 1.0 MB GLB; L_build_1/2/3 + Concrete House 003 ≤ 0.5 MB GLB total | | ≤ 2.0 MB embedded | These ship ONLY if the running total (tracked in the swap log, §11) stays ≤ 32 MB after adding them. They are the first cut. |

**Vertex budgets (Goldfish — a byte budget alone doesn't constrain geometry):** the Girl at
48,092 verts is ~3 MB of geometry alone (position + normal + UV + skin indices/weights ≈
64 bytes/vert) — she cannot fit a 0.9 MB GLB without decimating to roughly a third, which
the v1 doc never stated. Budgets: each human ≤18k verts (Girl 48k → ~16k; the 71k version
only AFTER a decimation test, not by eye — decide 48k vs 71k on the decimated result);
Hen ≤3k; cow ≤8k; dog ≤6k; bird ≤3k; Honey Locust ≤10k; each grass species ≤2k per plant;
each enterable building ≤30k; Norton ≤50k faces (already); Jeep ≤20k; Old Car ≤20k;
Military Truck ≤25k; each weapon ≤2k. Zane's decimation target: ≤18k verts from the 109 MB
source. Worker vertex count: state from the Blender inspection at §9 step 1 (2.6 MB FBX on
disk).

**Why the old table was wrong (Goldfish):** it summed to ~23.1 MB of GLB (≈30.8 MB base64)
against an "18 MB" total that never said GLB or base64 — 14.4 + 30.8 ≈ 45 MB, over the cap
by ~13 MB. The table above is the honest math. **Cut order if the running total approaches
32 MB:** (1) texture resolution, (2) NY skyline / L_build / Concrete House, (3) Zane —
drop him entirely, (4) one grass species. Never silently exceed 32 MB — report it.

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
   license (L_build and Concrete House especially).
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

1. **Tooling + spike tests:** tag the pre-build known-good commit FIRST (`git tag pre-integration-<date>` — the rollback anchor; see the rollback rule below). Blender extracted + verified (`blender --version`). **SPIKE-TEST THE LARGEST CONVERSIONS FIRST — Zane (109 MB FBX), then the Norton (404k faces, 119 objects) — before the Hen proves the pipeline.** Hen-first means the riskiest files are discovered late, after the pipeline is "proven" on the easiest file. Record peak RSS per model in `work/conversions.md`. **Abort rule:** if Zane cannot convert within RAM, drop Zane immediately per §10 item 2. The VM has ~7.9 GB total RAM and showed only ~344 MB available at the Goldfish check (other work running) — the builder checks free RAM and closes other work before starting Blender. `work/convert.py` written (step-1 deliverable). Blender inspection of Worker / Girl / Zane / Hen animation status (all four — the "unverified until Blender" rule, §3b) + Worker vertex count + Hen material→mesh mapping. **Flow interfaces defined BEFORE any placement work:** pickup kinds (POI restock, bandage kind, drop kind), HEALTH map category, discovery-toast rule (§6 items 1/6/9), surge trigger (§6 item 4), door E-slot (§5). `preview/model-preview.html` built + published + smoke-tested with the known-good chicken GLB (§8).
2. **Hen → chicken (`q6`/`t.scene`).** Smallest, safest first swap. Proves the pipeline end-to-end.
3. **Old Car 04 → wreck (`W6`/`c.scene`).** Simple static swap; also upgrades the stranger scene's backdrop.
4. **Worker + Girl + Zane → mobs.** The biggest visual win. Includes the `torso` proxies
   and procedural walk. Verify mob walk/chase/windup in a headless playthrough screenshot
   set (or a short scripted camera pass).
5. **Worker (seated) → Wounded Stranger.** Same model, baked seated pose.
6. **Shotgun / machete / flashlight → viewmodels** (when the files land). Verify the
   first-person screenshots (weapon in hand, muzzle position).
7. **Honey Locust → oak (`_6`/`r.scene`) + 4th tree variant; Maxtree grasses → instanced tufts.**
8. **Enterable buildings:** medieval house → windmill → village homes, each with doorway,
   colliders, shelter rect, door + knob + glow. Furnish pass per §4. Placement verification
   (top-down screenshot or collider dump, §4) BEFORE each building's swap commits. The
   truck-bed bandage pickup (§6 item 1) and the door E-slot (§5) are built with the
   buildings, not after them.
9. **Vehicles:** Jeep (the car Kes finds and drives — primary drivable + chase vehicle) →
   Norton (rideable) → Military Truck (roadblock + supply site). Vehicle POIs + discovery
   toasts (§6 item 6) are built with the vehicles, not in step 11.
10. **Fillers:** New York skyline, L_build_1/2/3, Concrete House 003 (only after
    origin/license confirmed).
11. **Progression + flow items (§6) — the REMAINING ones:** compass arrow, Day 4 surge, fall
    rebalance, opening toast, fight-or-flight reset, engine sound. (Supply restock/bandages/
    drops and vehicle POIs were built with steps 8–9 — the flow interfaces were defined in
    step 1, before placement. These are code edits in `index.html` — small, testable one
    at a time.)
12. **Pending downloads (cow / dog / bird / buildings / weapons) slot into steps 2–8 above
    as they land, per the §2b dependency table** — a swap whose file hasn't landed is
    skipped with a clearly-marked README TODO; all other swaps proceed. Do not hold the
    whole build for one file.

**Per-swap verification ("done" means ALL of these):**
- **License + source URL recorded in the §11 swap log BEFORE the commit (item 0).** No
  commit without them. The browser task's final report (or a page re-check) must be filed
  as a document before step 2 — the 8 landed models currently have NO recorded source URL
  or license, and the builder does not ship any model whose license is unclear.
  (Correction: "CGTrader free models are Royalty-Free unless the page says otherwise" is
  UNVERIFIED — CGTrader free models carry varying licenses, some editorial / attribution /
  non-commercial. Verify per model; do not assume.)
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
| 1 | **Worker / Girl / Zane may have NO baked walk/run animations.** The mob walk would then be fully procedural bone rotation, which can look stiff. | Verify in Blender FIRST (step 1 of the build) — all animation claims are UNVERIFIED until then. The plan is procedural bone rotation on RENAMED bones (§3b); baked clips are NOT the plan (no AnimationMixer for mobs). If the procedural walk looks stiff in the frame-sequence capture, bake a simple walk cycle in Blender from the rig (rotate thighs/calves/arms on keyframes, 1 s loop) — doable, but budget the time. |
| 2 | **Zane Ryder is 109 MB.** Decimation + texture resize may still leave it over budget, or lose the face detail. | Decimate in stages (50% → check → 25% → check). Textures to 1024 px, then 512 px if needed. Target ≤18k verts (see §7 vertex budgets). If it still won't fit 0.9 MB, DROP HIM ENTIRELY and use Worker/Girl only — using him for fewer mobs saves nothing (the GLB is embedded once regardless of how many mobs use it). |
| 3 | **The Girl is "stylized" by name.** Kester picked her personally — her pick stands. But verify she reads as a realistic person next to the Worker, not a cartoon. | Preview screenshot side-by-side with the Worker. If she reads cartoonish, use her sparingly (fewer female mobs) and tell Kester plainly. |
| 4 | **The Jeep may have no interior.** First-person driving needs one. | Verify on download. **Interior gate (§3b):** interior confirmed → first-person chase car; no interior → the Jeep is NOT drivable as a first-person vehicle — the fallback is a scoped chase-camera build (a new camera system, decided with Kester told plainly), NOT a silent "decide at build time". |
| 5 | **The Norton is 404k faces.** Decimation could mangle the chain/spokes. | Decimate the body aggressively, keep wheels/chain at higher density, verify in preview. If the chain won't survive, hide the chain (it's a detail, not a load-bearing part). |
| 6 | **Enterable interiors need furniture the downloads don't include.** | §4 furnishing plan: reuse village furniture if separable; else prop-library furnishing; flag "furnish pass 2" rather than blocking the build. |
| 7 | **The medieval block may be an unusable Lumion scene; the archviz scene is 491 MB.** | Skip both for this build (already decided in §4). No build time spent on them. |
| 8 | **L_build + Concrete House origins are unconfirmed.** | Do NOT ship either until the browser report confirms source URL + license. The other six landed models also have no recorded license yet — §9 item 0 blocks their commits until the browser report (or a page re-check) is filed. |
| 9 | **Single-file size.** 14.4 MB → ~32 MB target. Kester plays on a phone (DeX). | §7 budgets + the 32 MB hard cap. If the cap breaks, cut textures first, then models. |
| 10 | **The browser task hasn't finished.** Cow/dog/bird/buildings/weapons/vehicles may land late or not at all. | Build order (§9 step 12) lets ready swaps proceed; late files slot in. If a file never lands, the old model stays and the README says so — no silent gaps. |
| 11 | **The `house` (`p6`/`g.scene`) and `cottage` (`H6`/`B.scene`) GLB placements are unmapped.** | Builder traces their usage before touching them (§3c). The cottage GLB is the cottage wall/floor structure — it stays as-is in this build (see §4 known difference: cottages keep their position-zone shelter). Do not swap blind. |
| 13 | **Supply POI inventory is unverified.** §6 item 1 cites 5 of 11 POI positions from the bundle; the remaining 6 come from the build-start inventory of `world.pois`. | If the inventory finds a different count or values, §6 item 1 is restated from the inventory, not from the doc's examples. |
| 14 | **The Day 4 surge depends on the truck roadblock.** | If the truck swap (step 9) slips or the (0, -80) placement fails verification, the surge (§6 item 4) has no on-foot gate and must be re-scoped — it cannot ship as a drive-by. |
| 12 | **The chase stage is PROPOSED, not approved.** | No code. The design notes the dependency (vehicles) and stops there. |

---

## 11. Swap log (fill in as swaps land — also mirrored in README.md)

| Swap | Model | Source URL | License | Format in | GLB size | Pre-swap commit | Live verified |
|---|---|---|---|---|---|---|---|
| (none yet) | | | | | | | |

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
