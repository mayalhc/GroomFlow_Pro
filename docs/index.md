![GroomFlow01.jpg](assets/GroomFlow_B.png)

🇺🇸 English | [🇰🇷 한국어](./KO_index.md)


# 📖 GroomFlow Pro Add-on Guidebook

<div align="center">
  <iframe width="640" height="360" src="https://www.youtube.com/embed/3SJY1sfO5JY" title="GroomFlow PRO v1.6.0 Guide New Update" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
</div>

* **Welcome to GroomFlow Pro**
  * Welcome to the official documentation and user guide for GroomFlow Pro.
  * Learn how to maximize your grooming workflow using this advanced guide-driven hair system.
![GroomFlow_Pro_10.gif](assets/GroomFlow_Pro_10.gif)
---

## 🆕 What's New in v1.8.1 (Performance)

* **NEW: Pin the Tied Region during Hair Dynamics (Blender 5.2+)**
  * Only the stretch of hair a tie grips stays pinned now - not the whole
    curve. The pin peaks at full strength exactly where the tie grips and
    fades smoothly to nothing toward root and tip, so the free lengths
    either side keep simulating.
  * Each tie gets a **Pin Width** slider (select the tie to see it): 0.25
    holds the middle half of the strand, 0.5 holds everything but the very
    ends.
  * The solver's own state is never touched: the pin lives in the Hair
    Dynamics node wrapper as a per-point mask (`gf_pin`) written from the
    ties, so there is nothing to reset and no per-frame Python.
  * Toggle it with **Pin Tied Strands** in the Hair Dynamics settings.
    Untying a tie unpins its strands automatically. Grooms whose dynamics
    were attached before this update - or whose wrapper was wired by hand
    and broke - get the whole thing rebuilt from scratch by the new
    **Rebuild Dynamics** button (settings are kept, and the children's
    wrapper is rebuilt with it).
* **Fix: "Build Children" did nothing with the Live Engine off**
  * The one-shot build path stood down behind the live-engine guard, so with
    Live toggled off it created the children object and computed nothing.
    Build now always computes.
* **Generation is much faster**
  * The per-point Python loop that filled every strand (100k+ iterations on a
    5k-guide groom) is now a handful of numpy operations, and the evaluated
    pose is applied to the build mesh with `foreach_set` instead of two
    per-vertex loops.
* **Live engine ticks are lighter**
  * GPU input textures are re-uploaded only when their data actually changes
    (the RNG table and surface normals now upload ~never during playback),
    the guide-follow attributes are rewritten only when guide roots move,
    slider drags coalesce into one deferred rebuild instead of one per drag
    event, and the depsgraph update list is scanned once per pass instead of
    once per guide.
* **Scene-wide scans removed from the hot path**
  * Tie orphan cleanup now runs only when something was deleted (plus load
    and undo) instead of on every depsgraph update; the braid panel computes
    its guide distances once per redraw instead of three times; texture-mask
    generation no longer copies the entire image pixel buffer just to force
    an update; fur strand lengths use a cheap deterministic hash instead of
    building a random generator per strand.

---

## 🆕 What's New in v1.8.0

* **Editing a generation control no longer resets your groom**
  * This is the big one. Changing Guide Density, Length, Resolution - any
    generation setting - rebuilds the groom, and until now that rebuild threw
    away every comb stroke, every cut and every braid. Which meant that in
    practice you could not touch those controls again once you had started
    working.
  * **Keep My Edits** carries your work across the rebuild. Your offsets are
    lifted off before it and put back on after, so the groom is regenerated
    with the new settings and still combed the way you left it.
  * With the guide count unchanged they go back **exactly** - measured over
    twenty rebuilds with a braid running, the groom did not drift at all. When
    you raise the density, the new guides take the shape of their nearest
    neighbours, so they come out combed rather than standing straight up.
  * **Thickness does not rebuild at all any more.** Vertex Min/Max and Tip
    Thickness only write the strand radius, so they are applied in place -
    not a single point moves.
  * See **Section 12 → Keeping Your Work When Settings Change**.
<br>
<br>
* **Braid**
  * Braids the hair you have already combed. Put the 3D cursor where the braid
    should start, press **Add Braid**, and the strands that pass through that
    point are gathered into three bundles and woven around each other.
  * It works on the guides, so the children follow it the same way they follow
    combing — nothing about the way you build a groom changes.
  * **Knot Tightness** pinches the braid where the strands cross, which is most
    of what makes hair read as braided rather than as a rope.
  * **Tail Length**, **Tail Cinch** and **Tail Relax** tie the braid off and let
    what is left hang loose below the tie.
  * **Twist** and **Roundness** decide how round it looks. A plait is naturally
    flat — wide from the front, thin from the side — and these are what give it
    an even silhouette from every angle.
  * **Hold Under Simulation** keeps the braid woven while Hair Dynamics runs.
    Blender simulates every guide as an independent strand and nothing in the
    solver knows the three bundles are interlocked, so without this a braid
    comes apart a few frames in.
  * **Set 3 Clump IDs** gives the braided hair Clump IDs 1, 2 and 3 — one per
    strand of the plait.
  * See **Section 10 → Braid**.
<br>
<br>
* **Hair Cut**
  * Put a plane - or any object - where the hair should end and press **Cut**.
    Every strand that crosses it stops there.
  * A mesh cuts along its actual surface, so a plane only cuts the hair it
    covers, the way scissors do. Anything else cuts along its own Z plane.
  * **Jitter** breaks the cut line up, because a real haircut is not a laser.
  * See **Section 10 → Hair Cut**.
<br>
<br>
* **Make Groom Base** *(for MetaHuman and Unreal characters)*
  * One button duplicates the character's mesh as something you can actually
    groom on: the same surface in the same place, at scale 1, with no rig and
    no shape keys. Weights, UVs and materials come across; the original is left
    alone.
  * Blender's hair solver blows up on a rig scaled 0.01 - the same hair
    stretched 192x in twenty frames there and 1.00x on a base.
  * See **Section 1 → Make Groom Base**.
<br>
<br>
* **Mirror Weights, and painting both sides at once**
  * Blender's own symmetry cannot pair the vertices of a sculpted head, so it
    silently does nothing. GroomFlow pairs them itself, with a tolerance.
  * **Auto Mirror While Painting** copies each stroke onto the other side a
    moment after it lands.
  * See **Section 2 → Mirroring a Mask**.
<br>
<br>
* **Units — moved to the top of the panel**
  * It has to be settled before the first groom, and it used to be reachable only after the children were set up. It is at the top of the GroomFlow panel now.
  * **Guides, children, braid and texture masks all use the same unit now.** Previously only the children generator read this setting.
  * See **Section 1. Units & Scale**.
<br>
<br>
* **Root Clump**
  * A second clump profile that grips the hair near the scalp and lets go
    further down, on top of whatever the main Clump is doing.
  * Use it to close the roots of a groom whose lengths are meant to stay loose.
  * **Root Clump End** sets how far down the grip reaches.
<br>
<br>
* **Apply Simulation to Guides**
  * Blender's hair solver has no *apply*, so a pose you liked was only ever
    visible while the frame stood still. This writes the simulated shape into
    the guides.
  * From there it is an ordinary groom again — comb it, braid it, export it.
  * See **Section 9. Hair Dynamics & Collision**.

---

## 🚀 Key Features Guide

* **Guide-Based Strand Generation**
  * Creates dense hair strands from a lightweight guide curve workflow without the performance limitations of traditional systems.
  * Built around a guide-to-strand generation pipeline, allowing artists to design clean guide curves while automatically generating large amounts of production-ready strands.
![clumpid_01.png](assets/clumpid_01.png)
<br>
<br>
* **Surface-Locked Strand Distribution**
  * Fixes the issue where traditional duplication setups cause strands to float above the surface or clip through the mesh, especially around eyebrows, beards, eyelashes, and high-curvature facial areas.
  * Uses a BVHTree-powered surface projection system that snaps generated strand roots directly onto the target mesh.
  * Every generated strand remains accurately attached to the skin without floating roots, gaps, or penetration issues.
![clumpid_02.jpg](assets/clumpid_02.jpg)
<br>
<br>
* **Optimized Production Workflow**
  <br>
  * Keeps guide curves completely separated from generated output strands, providing a clean and non-destructive grooming workflow.
  * Mirrors modern professional grooming pipelines while maintaining a simple Blender-native workflow for character hair, eyebrows, facial hair, fur, and Unreal Engine groom preparation.
  * Designed to maximize visual fidelity while reducing manual cleanup and correction work during production.
![clumpid_03.jpg](assets/clumpid_03.jpg)
---

## 🛠 Installation Guide

Before starting the workflow, make sure to set up the add-on correctly by following these steps:

1. **Download**
   * Get the latest version of the GroomFlow_Pro add-on file ready.
<br>
<br>
2. **Installation**
   * Install and activate the program within Blender preferences.
   * Follow the general Blender add-on installation guide.
<br>
<br>
3. **Install Essential Extensions**
   * Go to the Extensions menu inside the add-on preferences.
   * Click 'Install' on the recommended base extensions to unlock full functionality.
<br>
<br>
4. **Preferences**
   * Adjust the options to fit your specific workspace layout.

---

## 1. Units & Scale

Every length in GroomFlow — strand length, thickness, child radius — is measured in metres, and the add-on converts them to whatever scale your character is actually built at. **This is automatic; there is nothing to set for a normal groom.**

* **Why it matters**
  * A character exported from Unreal is authored in centimetres and then scaled down by the rig, so one metre is 100 units in its local space. A strand length of `0.5` on such a character used to come out a hundred times too short, and the hair collapsed into a speck at the scalp.
  * GroomFlow reads the true scale off the character itself, so the same slider value gives the same real-world size on any rig.
<br>
<br>
* **Units** *(at the top of the GroomFlow panel)*
  * It sits above everything else because it has to be settled **before** the first groom, not found afterwards. Every length in the add-on — guides, children, braid, texture masks — is a world metre multiplied by this one number.
  * **Auto Detect** — the default. Reads the scale off the mesh the hair is bound to. Leave it here unless you have a reason not to.
  * **Manual** — type the factor yourself. Use this when your mesh has a deliberate non-uniform scale that is part of the look rather than a unit mismatch, and you do not want it corrected.
  * A line under the setting reports what was detected and which object it read, for example `1:1 (metres)` or `100x (Unreal / MetaHuman, cm)`.

> Blender's hair **solver** is the one thing that a scaled rig still breaks, and that is a limit of the solver rather than of the units — see **Section 9 → The rig's scale**.

> **Changed in v1.8.0:** Units moved from the Children panel to the top of the GroomFlow panel, and it is now one setting for the whole add-on. It used to be read by the children generator alone — the guide generator, the texture mask and the braid each measured their own scale off whichever object they happened to be holding, so **Manual** was ignored outside the children and a guide, its children and the head could each mean a different metre.

> Upgrading from v1.5? The old **Target Base Scale** slider is gone — this replaces it and needs no input. If you had set it to something other than `1.0` to compensate for a scaled character, that compensation is now handled for you.

---

### Make Groom Base

An Unreal-authored character is not a comfortable thing to grow hair on. Its
rig is scaled by 0.01, and Blender's hair solver reads the object's own units
as metres - so 5 cm of hair is simulated as a 5 metre strand and blows up
within a few frames. It is also deformed by an armature, so its shape changes
under you while you comb.

**Make Groom Base** duplicates it as none of those things: the same surface, in
the same place in the world, **at scale 1**, with no rig and no shape keys.
Weights, UVs and materials come across, so masking and weight painting work on
it. The original is untouched and hidden, and any groom already aimed at it is
re-pointed at the base.

It also puts the object's origin on the character's own midline, which is what
Blender's symmetric painting reflects about.

Measured on the same hair over twenty frames: 192x segment stretch on the 0.01
rig, 1.08x on the base.

---

## 2. Masking Method

Before generating hair, select which masking mode to use. The two modes are completely independent and each maintains its own separate layer list.

* **Vertex Weight**
  * Uses a vertex group painted on the mesh to control where and how long hair grows.
  * Paint red areas for full-length hair and blue areas for shorter hair or no hair.
  * Best for organic shapes like scalp hair, eyebrows, and beard regions.

![WeightPaint.png](assets/WeightPaint.png)

<br>
<br>
* **Texture Mask**
  * Uses a painted image texture to define the hair growth area and density.
  * Each layer has its own texture image, allowing multiple distinct hair regions to be managed independently.
  * Best for precise, UV-based control over hair placement — for example, fur patterns or stylized hair zones.

![TextureMask.png](assets/TextureMask.png)
---

### Mirroring a Mask

Blender's X Mirror pairs a vertex with whatever sits at exactly `-x`, within a
tight tolerance. A MetaHuman fails that twice over: it is sculpted, so the two
halves differ everywhere by a fraction of a millimetre, and its midline is not
at local `x = 0`. Measured on a mesh with 2 mm of asymmetry sitting 5 cm off
centre, Blender's mirror paired **0.4%** of the vertices - so it silently did
nothing. Symmetrize "works" by throwing half the character away.

GroomFlow pairs them on the data instead - it finds the mesh's own midline,
reflects each vertex about it, and accepts the nearest match within a
tolerance. Same mesh: **100% paired**. Only weights are written; the geometry is
never touched.

* **Center Origin on Midline**
  * Puts local `x = 0` on the character's midline without moving the mesh, and
    switches X symmetry on. Blender's symmetric painting reflects the brush
    about the origin, so an origin off the midline is why the mirrored cursor
    shows but only one side gets painted.
<br>
<br>
* **Auto Mirror While Painting**
  * Copies each stroke onto the other side a quarter of a second after it
    lands. Erasing mirrors too. Leave it off on a very heavy head and use the
    button instead - both use the same pairing, so you can mix them freely.
<br>
<br>
* **Mirror Weights**
  * Mirrors on demand: **+X to -X**, **-X to +X**, or **Both**, on the active
    group or all of them.
  * The **ⓘ** button reports how much of the mesh pairs at four tolerances, so
    a bad setting is visible rather than silent.

---

## 3. Generation & Masking Panel

This collapsible panel contains the vertex group list when using Vertex Weight mode.

* When a **mesh** is selected, the vertex group list for that mesh is displayed.
* Use **Add / Remove** buttons to create or delete vertex weight groups.
* The **Lock** button at the top of the list prevents accidental weight changes.
* In **Texture Mask** mode, a note directs you to the Texture Mask Hair Layers section below.

---

## 4. Hair Curve Layers

The Hair Curve Layers panel manages the list of generated hair objects in **Vertex Weight** mode.

* Each entry in the list represents one generated hair curve object linked to a vertex group mask.
* Use the **Up / Down** arrows to reorder layers in the stack.
* Use **Add** to create a new empty slot, or **Remove** to delete the selected layer entry.
* Clicking a layer in the list automatically activates the corresponding hair curve in the viewport.

---

## 5. Texture Mask Hair Layers

This panel manages hair layers generated using **Texture Mask** mode. It works independently from the Vertex Weight layer list.

* Each entry represents one hair curve object driven by a specific texture image.
* **Add** creates a new empty texture layer slot. The slot becomes active and ready to receive a generated hair object.
* **Remove** deletes the selected texture layer slot.
* **Texture Mask Settings** — shows the image picker for the currently active texture layer. Assign or create the mask image here before generating.
* **Invert Mask** — inverts the brightness values of the active mask image, flipping which areas grow hair.

### Texture Mask Workflow (Step by Step)

1. Switch Masking Method to **Texture Mask**.
2. Open the **Texture Mask Hair Layers** panel.
3. Press **Add** to create a new layer slot.
4. In the **Texture Mask Settings** area, assign or create a mask image for this layer.
5. Press **Go to Texture Paint Mode** to paint the mask directly on the mesh.
6. Return to Object Mode and press **Mask Generate** to create the hair curves.
7. Repeat from step 3 to build additional independently-masked hair layers.

> Each layer must have its own image assigned before generating. Generating without an image will use any existing untitled image or report a warning.

---

## 6. Strand Shape Controls

These settings control the shape and distribution of generated hair strands. Changing any value while a hair curve layer is active will automatically regenerate that layer in real time.

* **Lock / Unlock**
  * Locks the strand controls to prevent accidental changes after a groom is finalized.
  * Always lock before sculpting to protect your work from being overwritten by parameter changes.
<br>
<br>
* **Min Length**
  * Sets the minimum length for hair grown in low-weight areas.
  * Raising this value means even sparse areas produce longer strands.
<br>
<br>
* **Max Length**
  * Sets the maximum length for hair grown in fully-weighted (red) areas.
  * This is the primary length control for the densest regions of the groom.
<br>
<br>
* **Guide Density**
  * Controls the total number of hair guide curves generated on the surface.
  * Higher numbers produce denser coverage. Start lower while designing and increase for final output.
<br>
<br>
* **Spawn Radius**
  * Controls how far each generated hair root can randomly offset from the mesh surface sample point.
  * At 0.0, roots snap precisely to the surface. Increasing the value spreads roots into a wider area.
<br>
<br>
* **Weight Threshold**
  * Sets the minimum weight value required for a point on the mesh to spawn hair.
  * Raising this cuts off hair in lighter-weight zones. Lowering it below 0.01 ignores weights and spawns uniformly.
<br>
<br>
* **Strand Resolution**
  * Specifies the number of control points making up a single hair strand.
  * Higher values produce smoother, more flexible curves but increase memory and viewport load.
![GroomFlow_Pro_08.gif](assets/GroomFlow_Pro_08.gif)

!!! warning
    * **Never Modify Properties After Manually Sculpting Curves**
      * Changing any Strand Shape value forces a complete regeneration of the hair, overwriting all sculpt edits.
      * Always use the **Lock** button before entering Sculpt Mode to prevent accidental overwrites.

---

## 7. Thickness & Noise Settings

* **Min Root Thickness**
  * Sets the minimum thickness for the root area of generated strands, applied in low-weight zones.
<br>
<br>
* **Max Root Thickness**
  * Sets the maximum thickness for the root area of generated strands, applied in high-weight zones.
<br>
<br>
* **Tip Thickness**
  * Controls the thickness at the very tip of each strand. Set near 0.0 for a natural tapered look.
<br>
<br>
* **Frizz Noise Strength**
  * Adds random directional noise to each strand, creating a naturally messy or frizzy appearance.
  * Higher values produce more chaotic, irregular silhouettes.
![GroomFlow_Pro_09.gif](assets/GroomFlow_Pro_09.gif)

---

## 8. Hair Style Nodes Stack

Attach Blender geometry node modifiers to the active hair curve to shape the final look. Each button loads the corresponding node group from Blender's built-in Hair Essentials library.

* **Add Clump**
  * Pulls groups of strands together toward common cluster points, creating natural hair clumping.
<br>
<br>
* **Add Frizz**
  * Adds high-frequency noise breakup to individual strands for a naturally rough or frizzy texture.
<br>
<br>
* **Add Interpolate**
  * Fills in additional strands between existing guides, dramatically increasing visible density.
<br>
<br>
* **Add Duplicate**
  * Scatters offset copies of each strand to build up volume quickly without adding new guides.
<br>
<br>
* **Add Braid**
  * Weaves strands into a braided rope pattern for stylized braid or twist effects.
  * This is one of Blender's own style nodes. For GroomFlow's own braid, which
    works on the guides and survives simulation, see **Section 10 → Braid**.
<br>
<br>
* **Add Curl**
  * Applies a helical curl deformation along the length of each strand for curly or wavy hairstyles.
![GroomFlow_Pro_07.gif](assets/GroomFlow_Pro_07.gif)

---

## 9. Hair Dynamics & Collision

> Requires **Blender 5.2 or newer**. On older versions the panel says so and the buttons stay inactive.

Blender 5.2 introduced a new XPBD physics solver for hair. This panel wires it up for you and puts its settings where you can reach them — otherwise every adjustment means opening the node editor and digging inside a node group.

### Buttons

* **Hair Dynamics**
  * Select a guide curve and press this. The solver is attached and set to collide with the mesh the hair is attached to.
  * Press **Play** on the timeline to see it simulate.
<br>
<br>
* **Collider**
  * Select any mesh the hair should bump into — the body, a shoulder, a hat — and press this.
  * The scalp itself does not need a collider; that is already handled by **Surface Collision** below.
<br>
<br>
* **Apply Simulation to Guides**
  * Writes the shape the simulation has reached into the guide curves, as if you
    had combed it there. Blender's hair solver has no *apply*, so a pose you
    liked was only ever visible while the frame stood still.
  * It can switch the solver off and return to the start frame afterwards, so
    the groom holds that shape instead of falling again.
  * It refuses when a modifier has resampled the curves, because writing that
    back would scramble the groom.
<br>
<br>
* **Remove Dynamics**
  * Strips the dynamics and collider setup back off the selected objects.

### Settings

These appear once Hair Dynamics has been added, and update the simulation as you change them.

* **Physics**
  * On, the solver simulates. Turn it off and the hair simply follows the animation without simulating.
<br>
<br>
* **Solver** — *Substeps*, *Constraint Steps*
  * Raise these if the hair passes through the body or stretches under fast motion. Higher values cost more to compute.
<br>
<br>
* **Structure**
  * **Bendiness** — how freely the strand bends. Low is stiff hair, high is soft.
  * **Root Bendiness** — how much it may bend right where it meets the scalp.
  * **Stretchiness** — how much the strand may lengthen. Keep near zero for hair.
  * **Mass**, **Friction** — weight and how much the strands drag on each other.
  * **Linear Damping** — raise it if the hair jitters.
  * **Angular Damping** — raise it if the hair whips around too freely.
<br>
<br>
* **Collision**
  * **Surface Collision** — keeps the hair off the mesh it grows from.
  * **Deforming Surface** — tick this when that mesh is rigged or animated.
  * **Surface Friction** — how much the hair slides across it.
  * **Gravity** — the hair falls. Turn it off for stylized or zero-gravity looks.

### Recommended Workflow

1. Shape your guide curves first. Dynamics simulates whatever shape you give it.
2. Select the guide curve → **Hair Dynamics**.
3. Select the body mesh → **Collider**.
4. Play the timeline and watch. If hair passes through the body, raise **Substeps**; if it jitters, raise **Linear Damping**.
5. Leave **Live OFF** while previewing — the children follow the simulated guides on their own (see Passive Follow in the next section).

### The rig's scale, and why the simulation explodes on a MetaHuman

Blender's hair solver runs in the curve object's **own local space**, and treats
those units as metres. On a character whose rig is scaled — a MetaHuman or any
Unreal-authored figure is authored in centimetres and scaled by 0.01 — the same
5 cm of hair is 5 units long in that space, so the solver is trying to simulate
what it believes is a 5 metre strand. It diverges within a few frames, and the
guides shoot out of the head in straight spikes.

Measured on identical hair, in the identical place in the world, over twenty
frames:

| Rig | Worst segment stretch | Result |
| --- | --- | --- |
| 1:1, metres | 1.12x | stable |
| scaled 0.01 (MetaHuman / Unreal) | **192x** | explodes |
| scaled 0.01, hair object left at scale 1 | 1.00x | stable |

It is **not** gravity, **not** collision and **not** GroomFlow's Units setting —
with gravity and collision both switched off it still reached 43x, and raising
Substeps from 10 to 100 did not help either. The only thing that matters is the
world scale of the object the solver runs on.

**What to do**

* Apply the rig's scale (`Ctrl+A → Scale`) before simulating, or
* keep the hair curves object at world scale 1, so it follows the head without
  inheriting its scale.

The panel warns you when the groom's world scale is not 1 and tells you what it
measured, so you are not left guessing at the spikes.

> This is a property of Blender's solver, not of GroomFlow. Everything else in
> the add-on — generation, children, braid, export — already works correctly on
> a scaled rig; see **Section 1. Units & Scale**.

---

## 10. Realtime Simple Children

The Children system generates a cloud of short, naturally distributed child strands around each guide curve. Children stay synchronized with the guide in two complementary ways — a lightweight always-on follow, and an optional precise live recompute — so you get real-time feedback without paying the full performance cost all the time.

As of v1.6.0 the live recompute runs on your graphics card, roughly ten times faster than before. A status line at the top of the panel tells you which path is active:

* `GPU: OPENGL / <your graphics card>` — the fast path is running.
* `GPU: unavailable - using CPU` — GroomFlow checked your GPU, the result did not match, and it switched to the CPU. The groom is still correct, just slower.

To use this panel, select a **hair curve object** in the viewport. The panel will display the active layer name and its settings.

### Setup

* **Active Layer**
  * Displays the name of the currently selected guide curve receiving children.
<br>
<br>
* **Target Mesh**
  * Select the body or scalp mesh that this guide curve is attached to.
  * Use the eyedropper to pick the mesh directly from the viewport.
  * This must be set before building children.

### Buttons

* **Build Children**
  * Generates or regenerates children for the currently active guide curve.
  * Creates a `{CurveName}_Children` object in the scene containing all child strands, already surface-bound to the Target Mesh.
  * Also attaches a lightweight follow modifier that keeps children glued to whatever the guide is currently doing — see **Passive Follow** below.
  * Press this once per guide curve. Press again to rebuild if Child Count, Radius, Clump, or Root Distribution settings change.
  * Each guide curve has its own independent children object.
<br>
<br>
* **Live OFF / Live ON** (toggle)
  * Toggles the precise real-time recompute engine used while actively sculpting or combing.
  * **Live ON** — clump, twist, and root-scatter are fully recalculated every time the guide changes. Use this while you are actively shaping the groom with sculpt tools.
  * **Live OFF** — the recompute engine stops running. Children are not frozen, though — see below.
  * The Live Engine is shared across all guide curves. Turn it on while grooming, off when you're done shaping and just want to preview motion or simulation.

### Passive Follow — children keep moving even with Live OFF

Every time **Build Children** runs, a small geometry-node modifier (`GF_ChildrenGuideFollow`) is attached to the children object. It reads the guide's current position directly from Blender's own dependency graph and offsets the children to match — no Python code runs for this, so it costs virtually nothing.

This means: if you turn on Blender's native **Hair Dynamics** simulation on the guide curve, or simply play back an animation that poses the character, the children will follow along even with Live OFF. This is the intended way to preview simulation or animation — leave Live OFF, let Passive Follow carry the motion, and only switch Live ON again when you want to re-shape the clump/twist detail by hand.

> Passive Follow moves each child strand rigidly along with its guide's root — it does not re-run the clump/twist math. For that, use Live ON.

### Recommended Workflow for Multiple Guide Curves

1. Select guide curve 1 → set Target Mesh → press **Build Children**.
2. Select guide curve 2 → set Target Mesh → press **Build Children**.
3. Continue for all guide curves.
4. While actively shaping the groom, press **Live ON** — clump/twist recompute live as you sculpt.
5. When you're done shaping, press **Live OFF**. Children stay put, and Passive Follow keeps them attached if the guide is later simulated or animated.

> **Important:** Build Children and Live Engine are separate actions. You can build children for all curves first, then enable Live once. You do not need to turn the engine on and off between each curve.

![GroomFlow_Pro_04.gif](assets/GroomFlow_Pro_04.gif)
  <br>
![GroomFlow_Pro_05.gif](assets/GroomFlow_Pro_05.gif)

### Child Strand Settings

* **Child Count**
  * Number of child strands generated per guide curve.
  * Higher values produce denser hair volume. Balance against viewport performance.
<br>
<br>
* **Child Point Resolution**
  * Number of control points per child strand. Higher values make smoother curves.
<br>
<br>
* **Guides Inside Children**
  * On by default. The guide curves are carried inside the children object as well, marked so Unreal can tell which strands are guides.
  * This is what lets you export the children object on its own and have the complete groom — guides included — arrive in Unreal as a single asset. With it off, the guides and children stay two separate objects and a groom has to be exported twice.
<br>
<br>
* **Radius**
  * The spread radius around the guide root where child roots are distributed.
  * Small values keep children tightly grouped; large values spread them wide.
<br>
<br>
* **Length Min**
  * Minimum length ratio of child strands relative to the parent guide. Values below 1.0 create naturally varied shorter hairs.
<br>
<br>
* **Length Max**
  * Maximum length ratio relative to the parent guide. Values above 1.0 allow some children to extend beyond the guide tip.

![GroomFlow_Pro_06.gif](assets/GroomFlow_Pro_06.gif)

### Clump Settings

Controls how child strands pull toward the guide curve's tip, forming natural hair clumps.

* **Clump** — overall strength of the clumping pull toward the guide tip.
* **Clump Start** — the point along the strand where clumping begins (0 = root, 1 = tip). Below this the children stay fully spread.
* **Clump Shape** — how the clumping eases in between Clump Start and the tip. Low values pull the strands together early; high values hold them apart and close near the tip.
* **Clump Noise** — adds random per-strand variation to break up uniform clumping patterns.
* **Clump Twist** — rotates child strands around the guide axis as they clump, creating a spiral wrap effect. Negative values twist the other way.
* **Clump Offset** — shifts the point the strands converge on away from the guide tip.

> **Changed in v1.6.0:** Clump Start used to reach *full* clumping by the point it names, rather than starting there — which squeezed the whole transition into the first segment or two and left a visible hook at the root. It now behaves as described above. Grooms saved with a Clump Start above 0 will look a little looser than before; raise **Clump** slightly to match.

### Root Distribution Settings

Controls where child strand roots are placed relative to the guide root.

* **Root Spread** — radius of the disk area around the guide root where children are scattered. At 0.0, all children start exactly at the guide root.
* **Spread Along Guide** — when Root Spread is greater than 0, this stretches the scatter along the direction the guide lies in, instead of an even circle. Use it for hair that is combed flat against the scalp.
* **Root Seed** — random seed for the child root placement pattern. Change this to get a different arrangement without changing any other settings.

> **Fixed in v1.6.0:** *Spread Along Guide* previously had no effect no matter what it was set to. It now works as described.

### Clump ID

Every strand carries the ID of the clump it belongs to, stored in the hair data itself as `clumpid`. Unreal and other groom tools read this to keep a tuft of hair together when shading and simulating.

GroomFlow knows exactly which guide each child grew from, so its clump grouping is exact. Tools without that relationship have to guess it by clustering root positions, which splits real tufts apart and merges unrelated ones.

* **Clump Size**
  * How many neighbouring guides share a single clump.
  * At `1` — the default — every guide is its own clump. Raise it to merge nearby guides into larger, chunkier tufts.
  * Guides are grouped by where they actually sit on the head, not by the order they were created, so a clump is always a real physical tuft.
<br>
<br>
* **Clump Seed**
  * Reshuffles which guides fall into which clump. The number of clumps stays the same.
  * **This does not regenerate your groom.** Combing, sculpting and every strand position are left exactly as they are — only the grouping changes — so you can keep trying arrangements on a finished groom.
<br>
<br>
* **Root Clump**
  * A second clump profile that grips the hair near the scalp and lets go
    further down, on top of whatever the main **Clump** is doing. Use it to close
    the roots of a groom whose lengths are meant to stay loose.
<br>
<br>
* **Root Clump End**
  * How far down the strand the root grip reaches.
<br>
<br>
* **Preview Clump IDs** *(toggle)*
  * Colours every strand by the clump it belongs to and switches the viewport to Material Preview so you can see it.
  * Press it again to turn it off. The preview colour, its material and your viewport shading are all put back the way they were; the clump data itself is untouched.

![GroomFlow_Pro_06.gif](assets/GroomFlow_Pro_06.gif)

---

### Hair Cut

Cut the hair the way you would with scissors: put something where it should
end, and press **Cut**.

1. **Add Cutting Plane** makes a wireframe plane sized to your groom and picks
   it as the cutter. Any object already in the scene can be used instead.
2. Move and rotate it to where the hair should stop.
3. **Cut**.

* **Cut With** — any object. A mesh with faces can cut along its real surface;
  anything else - an empty, a plane you have not applied - cuts along its own Z
  plane.
* **Reads** — *Surface* cuts where the strand actually crosses the cutter, so a
  plane only cuts the hair it covers. *Plane* extends the cutter's Z plane
  forever, for a straight cut right across a groom.
* **Jitter** — breaks the cut line up per strand. A real haircut is not a laser,
  and a little of this is what stops the ends reading as one flat edge.
* **Keep At Least** — no strand is cut shorter than this. A root left with no
  length disappears from the groom and takes its children with it.

The strand keeps its point count and is re-spaced over what is left, so
children, clump ids, the braid and the export set all keep working. Cutting is
undoable, and it only touches the strands you have selected in Sculpt mode -
or all of them, when nothing is selected.

* **Set Guide Resolution**
  * A cut shortens a strand but leaves it with the same number of points, so
    the tip end can come out coarse. This re-spaces the guides over a new point
    count **without regenerating** - shape, combing, cuts, clump ids and the
    surface binding all survive. Child Point Resolution is raised to match.

---

### Braid

Braids the hair you have already combed, rather than generating three tubes for
you to place. The braid lives on the **guides**; children follow it through the
ordinary generation path.

**Making one**

1. Put the 3D cursor where the braid should start — usually the nape, or
   wherever the hair gathers.
2. Press **Add Braid**. Every strand that passes within **Gather Radius** of
   that point is taken into the braid, and the settings are measured off your
   groom rather than guessed.
3. **Update Braid** re-solves it. **Clear** puts the hair back and hands it over
   to you; the **X** removes the braid from the list.

> Brushing and combing a braided groom is fine. The braid records what it wrote,
> so your strokes survive and the weave re-forms around them.

**Gather**

* **Gather Radius** — how much hair the braid takes. The panel says how many
  guides that is, and how far the nearest one is when it reaches none.
* **Gather Falloff** — softens the edge of the gather, so the slider moves
  continuously instead of taking whole guides at a step, and so braided hair
  does not sit hard against untouched hair.
* **Center** — where the braid starts. It snaps onto the nearest strand when the
  braid is created.

**Shape**

* **Weave** — *Braid* is the three-strand plait. *Rope* is three helices twisted
  together. *Flat* keeps the three on one line that turns as it goes down.
* **Braid Diameter** — how far the bundles swing apart.
* **Inner / Outer Diameter** — how thick a bundle is at its narrowest and at its
  widest point. The difference between the two is the scallop along the braid's
  outline.
* **Bulge Bias** — how sharply the thickness changes between those two. At 0 the
  bundle keeps its Outer Diameter the whole way and there is no scallop.
* **Clump Hold** — how tightly hair clings to its own lock. 0 fills the bundle
  evenly; 1 collapses each lock onto its guide.
* **Guide Attract** — pulls the children onto the guides, for a clean, tidy
  braid.
* **Knot Tightness** — squeezes the braid where the strands cross. The weave
  already narrows there; this deepens it until each crossing reads as a knot.
* **Weave Depth** — adds to or takes from how far the bundles pass in front of
  and behind each other. 0 is the depth a plait has on its own; the bottom of
  the range flattens it into one plane.
* **Roundness** — lifts the bundles off the braid's centre line. At 0 the middle
  bundle passes straight through the centre, where it has no distance from the
  axis at all and Inner, Outer and Bulge cannot reach it — so only the outer two
  get shaped.
* **Twist** — turns of the braid's own plane over its length. A plait is flat, so
  without this it is wide from the front and thin from the side. Twisting makes
  every viewpoint the same.
* **Lock Flatness** — cross-section squash of each lock. This is the shape of the
  hair itself; use Weave Depth for how deep the braid is.
* **Roll** — rotates the braid's face around its own length. 0 faces away from
  the head.

**Along the braid**

* **Pitch** — the length of one full weave cycle. Smaller is a finer braid.
* **Braid Start** — where along the strands the weave begins.
* **Ease In** — how gradually it takes hold at that point.
* **Tip Scale** — how much of the braid's width is left at the tip. 0 closes it
  to a point.

**The tail**

* **Tail Length** — how much of the braid, measured back from the tip, is left
  loose below the tie. 0 braids all the way down.
* **Tail Cinch** — how hard the tie squeezes the hair where the braid ends.
* **Tail Relax** — how loosely the tail hangs. 0 keeps it gripped as a tight
  tassel, 1 opens it back to the braid's own width. Tied hair stays tied either
  way.

**Loose hair**

* **Flyaways** — the share of strands held back from the weave.
* **Flyaway Length** — how far those strands stand out.
* **Seed** — reshuffles which strands those are.

**Simulation**

* **Hold Under Simulation** *(on by default)* — keeps the braid woven while Hair
  Dynamics runs. The simulation still swings the braid; it just cannot pull the
  plait apart.

**Clump IDs and resolution**

* **Set 3 Clump IDs** — gives the braided hair Clump IDs 1, 2 and 3, one per
  strand of the plait. Hair outside the braid that already used those IDs is
  moved off them as a whole group. Changing **Clump Size**, or regenerating the
  guides, overwrites this — press it again afterwards.
* **Add Points for This Pitch** — appears when the guides are too coarse for the
  Pitch you asked for. A weave finer than the guides can carry does not come out
  coarse, it comes out wrong: narrow, flat and closer to a screw than a braid.
  This resamples the guides to the resolution the weave needs and raises Child
  Resolution to match.

---

## 11. Generation Options

* **Replace Existing Hair**
  * When enabled, generating hair overwrites the curves in the currently active layer.
  * When disabled, each generation creates an entirely new layer on top of existing ones.
  * Leave this enabled during normal grooming to avoid accumulating redundant objects.
![GroomFlow_Pro_03.gif](assets/GroomFlow_Pro_03.gif)
<br>
<br>
* **Generate on Vertices**
  * Snaps and generates hair guide curve roots precisely onto mesh vertices instead of face surfaces.
  * Useful for low-poly assets or grooms that require roots to align exactly with the mesh topology.
![GroomFlow_Pro_02.gif](assets/GroomFlow_Pro_02.gif)

---

## 12. Process Control Buttons

### Keeping Your Work When Settings Change

Changing a generation setting rebuilds the groom. It always has - and a rebuild
used to throw away every comb stroke, every cut and every braid.

* **Keep My Edits** *(on)*
  * Your offsets are lifted off before the rebuild and put back on after it, so
    the groom is regenerated with the new settings and still combed the way you
    left it.
  * When the guide count is unchanged, they go back exactly - measured over
    twenty rebuilds with a braid on, the groom did not drift at all.
  * When the guide count changes, each **new** guide takes the offsets of the
    **nearest old guide**, sampled along its own length, so the new hair comes
    out combed rather than straight.
<br>
<br>

> **Worth knowing:** because a new guide copies its nearest old one, taking
> **Guide Density down to 1 and back up again fills the whole groom with that
> one guide's shape.** There is only one guide left to copy from. It is not a
> fault - it is what "take the nearest one" means when there is only one. If
> you want the groom back as generated, turn **Keep My Edits** off for that one
> rebuild, or press Generate with it off and on again afterwards.

* **Auto Regenerate** *(on)*
  * Rebuilds as soon as a setting changes, the way it always has.
  * Turn it **off** on a heavy groom: the setting is remembered, the panel says
    the groom is behind, and **Generate** is what rebuilds. Nothing is lost
    either way - this is about when the work happens, not whether it survives.
<br>
<br>
* **Thickness never rebuilds at all.** Vertex Min/Max and Tip Thickness only
  write the strand radius, so they are applied in place - not a single point
  moves, whatever the two switches above are set to.

### Vertex Weight Mode

* **Weight Generate**
  * Runs the hair generation algorithm using the active vertex weight group as the mask.
  * Generates hair strands directly onto the mesh surface based on painted weight values.
  * Red areas produce full-length hair; lower-weight areas produce shorter or no hair.
<br>
<br>
* **Go to Weight Paint Mode**
  * Switches the active mesh into Weight Paint Mode for direct vertex weight editing.
  * Paint red (weight 1.0) for dense, full-length hair and blue (weight 0.0) to suppress hair.
<br>
<br>
* **Smooth Weights Gradient**
  * Softens sharp transitions in the active weight map into a smooth gradient.
  * Prevents abrupt length changes at the boundary between painted and unpainted areas.

![GroomFlow_Pro_01.gif](assets/GroomFlow_Pro_01.gif)
![GroomFlow_Pro_01_01.gif](assets/GroomFlow_Pro_01_01.gif)

### Texture Mask Mode

* **Mask Generate**
  * Generates hair using the active texture layer's image as the mask.
  * Bright (white) areas in the image produce hair; dark (black) areas suppress it.
<br>
<br>
* **Go to Texture Paint Mode**
  * Switches the active mesh into Texture Paint Mode with the active mask image pre-selected.
  * Paint white to add hair and black to remove it.

---

## 13. Snap Settings

* **Add Snap**
  * Applies a Geometry Nodes-based precision snap system to the active hair curve.
  * Keeps hair roots locked to the mesh surface, preventing floating or clipping even when the mesh deforms (shape keys, simulations, etc.).
  * Requires a hair curve to be selected. The Target Mesh must be set in the Children panel first.

![GroomFlow03.png](assets/GroomFlow03.png)

---

## 14. Unreal Engine Pipeline & Expert Synergy Workflow

Finalize your asset data blocks and prepare your grooms for cinematic export integration.
<br>
<br>
* **Scale — now handled for you**
  * Blender and Unreal describe world scale differently, which is why imported hair so often arrives as giant sheets or microscopic pins.
  * As of v1.6.0 there is nothing to calibrate. GroomFlow reads the real scale off your character and builds the groom at the correct size on both a metre-based Blender character and a centimetre-authored Unreal/MetaHuman rig. See **Section 1. Units & Scale**.
  * If a character has a deliberate non-uniform scale you want preserved, switch **Units** to **Manual** and enter the factor yourself.
<br>
<br>
* **Export the children object alone**
  * Leave **Guides Inside Children** on. The guides travel inside the children object, flagged as guides, so Unreal receives one complete groom instead of two half ones.
  * The strands also carry **Clump ID** and their guide assignment, so Unreal's clump-based shading and simulation have the grouping GroomFlow actually used rather than a re-guessed approximation.
<br>
<br>
* **Professional GroomForge Pro Synergy**
  * Exporting hair meshes can be tricky due to world matrix mismatches across different software spaces.
  * `Seamless Matrix Synchronization`:
    * When paired with the **GroomForge Pro** add-on, your separated guide and child strand layers bypass all transformation conversion issues entirely.
    * Ensures a 100% flawless, automated import layout into Unreal Engine's native Groom asset system without manual repositioning.
<br>
<br>
* **Performance Tip (Resolution Management)**
  * Optimize your setup dynamically to maintain professional-grade viewport reactivity while working on dense hair grooms.
  * `Real-time Efficiency`:
    * Lower the Strand Resolution down to `4` or `6` while designing massive hairstyles to maximize your viewport FPS smoothness.
    * Simply bump it back up to `12` right before rendering final frames in Cycles or executing your absolute groom exports to Unreal Engine.
