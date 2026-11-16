# Wonderland

**Author:** Zeynep Kesim

*Developed for the Computer Graphics Project course (MTAT.03.328), University of Tartu, Fall 2026.*

## Project Description

Wonderland is an interactive project inspired by surreal and dream-like environments. The user can explore the environment and interact with different objects. As the user interacts with them, the objects can change their appearance, movement, or behaviour, gradually making the world feel different.

The project will explore different ways of creating these changes through animation, lighting, materials, shaders, and particle effects.

## Goals

- Create an interactive environment
- Implement interaction with objects
- Create animated and visually changing objects
- Experiment with shaders, materials, lighting, and particle effects
- Create several surreal environmental transformations
- Produce a polished and visually interesting final result

## Expected Result

An interactive scene where the user can explore the environment and interact with objects.

## Technologies

- Unreal Engine 5 (Blueprints, Niagara)
- Blender
- Git and Git LFS
- GitHub

## Milestone 1 (06.10)

**Goal:** Set up the basic environment and scene.

- Create the Unreal Engine project and set up the basic scene ~ 1h
- Create a simple environment and add a few objects ~ 1.5h
- Set up the camera and basic navigation ~ 1h
- Set up the Git repository and project structure ~ 0.5h

**Bonus Tasks:**

- Create a basic interaction prototype
- Implement object interaction using Blueprints

### Development Notes

#### Scene and Atmosphere

I started from Unreal's First Person template (Blueprint) and built my own level, `L_Wonderland_Main`. The scene is a room open to the sky, with a doorway leading out into the clouds. A low sun shines in through the doorway, and volumetric fog turns its light into visible shafts. Sky atmosphere, volumetric clouds and a post-process volume (bloom, vignette) complete the soft, dream-like look.

#### Environment

Most of the environment is still a blockout made from basic shapes: the walls, an armchair, a window, a mirror, and clouds piled around and inside the room. The goldfish are an exception. I modelled a simple low-poly goldfish in Blender (body, tail and dorsal fin), exported it as FBX and imported it as a static mesh. Small goldfish float around the room, and one giant fish hangs outside the doorway to play with scale.

#### Materials

Everything uses a single master material, `M_Dream`. It blends between two colours (`ColorA` and `ColorB`) using a `Transform` parameter from 0 to 1, and also has parameters for glow, pulsing, roughness and metallic. Each object type has its own material instance with its own palette: turquoise walls, moss-green floor and armchair, orange fish, white clouds, a glowing window and a silver mirror. The second colour of each instance is the colour the object turns into when it transforms.

#### Movement and Camera

I tuned the character to feel slower and lighter, closer to moving in a dream: lower walk speed and gravity, more control in the air, a softer stop, and a slightly narrower field of view.

#### Interaction

Pressing **E** casts a line trace 5 metres forward from the camera. If the object it hits implements the `BPI_Interactable` Blueprint Interface, its `Interact` event is called. Only the object being looked at receives the message.

`BP_Interactable_Base` handles the transformation:
- On start, it creates its own dynamic material instance, so each object can change independently of the others.
- On `Interact`, a 1.5-second timeline drives the material's `Transform` parameter from 0 to 1 and scales the object up. Interacting again plays it in reverse.

Two child Blueprints use this base: `BP_Interactable_Fish` (goldfish turn gold and grow) and `BP_Interactable_Cloud` (clouds turn sky blue).

## Milestone 2 (20.10)

**Goal:** Make the world respond to the player's attention.

- Replace the E key with gaze-based interaction, so objects transform when the player looks at them for a moment ~ 2h
- Make the whole world change as more objects transform, with fog and light shifting gradually ~ 2h
- Make the goldfish swim slowly around the room ~ 1h
- Replace the armchair and the window frame with proper models, using free assets or Blender ~ 1.5h

**Bonus Tasks:**

- Turn the window into a view of another place
- Add a particle effect when an object transforms

### Development Notes

#### Looking Instead of Pressing

The E key is gone. Now you just look at something for about a second and it changes.

Every frame, the character casts a sphere trace (40 cm radius, up to 5 metres) from the centre of the camera. I switched from a line trace because a thin line kept slipping off objects, which made it almost impossible to keep looking at one thing for a whole second. If the trace keeps hitting the same interactable object, a timer counts up. Look away or at something else and it resets. Once it reaches `GazeDuration` (1 second by default), the object gets the same `Interact` message as before. So the transformation itself didn't change at all, only what triggers it.

I also made transformations one-way. Toggling back and forth made sense with a key, but with gaze an object would flip back if you just kept staring at it, which felt confusing. Now once something changes, it stays changed.

#### The World Slowly Changes

`BP_DreamDirector` is an invisible actor that keeps track of how deep into the dream you are. When the game starts, it counts every actor that implements `BPI_Interactable` and saves the starting fog values. Every time an object transforms, it lets the director know. The director works out how many objects have changed so far (say 3 out of 7) and moves the whole world towards a "deeper" look over three seconds.

That deeper look comes from two things. The height fog gets thicker and shifts from pale turquoise to pink and purple, and a second post-process volume (`PPV_DeepDream`) fades in, changing the colour temperature, tint, bloom and vignette. Everything goes through one function, `ApplyWorldState`, which takes a single value from 0 to 1. At 0 the room looks like it did in Milestone 1, at 1 every object has transformed.

#### Swimming Goldfish

The goldfish now swim in slow circles around the spot where they were placed, bobbing gently up and down and always facing the way they're going. Each fish starts at a random point on its circle, so they don't all move in sync. Radius, speed and bob height can be set per fish, so the giant one outside can move slower and in a wider circle. It still works with gaze, so you can follow a fish with your eyes until it turns gold.

#### New Armchair, Window and Mirror

The block armchair is now *GreenChair_01* from Poly Haven. Its green fabric fit the palette so well that I kept the original textures.

For the window and the mirror I used *Ornate Mirror 01*, also from Poly Haven. The model came with a single material for both the frame and the glass, so I opened it in Blender and moved the glass faces into their own material slot. Now the same gilded frame shows up twice on the wall: in the window the glass glows (`MI_Window`), and in the mirror it's a smooth silver surface that actually reflects the room (`MI_Mirror`).

#### Sparkles

When an object starts to transform, a small burst of glowing sparks appears around it. It's a Niagara system (`NS_TransformBurst`) made from one of the burst templates, with lower gravity and slower particles so they float instead of falling. It's attached to the object's mesh and scales with it, so it follows a swimming fish and gets bigger on the giant one.

## Milestone 3 (03.11)

**Goal:** Fill the room and turn the placeholder clouds into real ones.

- Furnish the room with more objects using free assets or models made in Blender ~ 2h
- Model soft, puffy clouds in Blender and replace the sphere clouds with them ~ 2h
- Give the clouds a soft material that glows slightly at the edges ~ 1h
- Add a soft glow that builds up while the player looks at an object, so it's clear what is about to change ~ 1h

**Bonus Tasks:**

- Finish the window view of another place
- Make some of the new furniture interactable with its own transformation, for example, books floating up from the shelf

### Development Notes

#### A Fuller Room

The room now has a sofa, a wooden drawer cabinet and a tree growing in the middle of it, all free models from Poly Haven. I tinted the sofa's fabric with a colour parameter in its material, so it's now a green very close to the armchair's and the two feel like a set. And the floor is no longer a flat colour. It's now grass with scattered stones, using a terrain texture from Poly Haven.

A few things took longer than expected:
- None of the Poly Haven textures connected to their materials on import. Part of the reason was that the unzipped folders were named like `Sofa_01_1k.fbx`, and Unreal couldn't build a path from a folder name ending in `.fbx`. After renaming the folders, I still connected the textures by hand (base colour, normal, roughness), setting the normal map compression and turning off sRGB for roughness and metal maps.
- The drawer came in as seven separate meshes (the body and six drawers), so I re-imported it with *Combine Static Meshes*.
- The tree's box collision covered its whole crown. It blocked the doorway and also stopped the gaze trace from reaching the fish behind it, so I removed it. The leaves use a masked, two-sided material with the alpha texture as the opacity mask.

#### Puffy Clouds

The sphere clouds are gone. I made three clouds in Blender with metaballs: a big puffy cluster, a long flat one and a tall one that builds up like a tower. Each one has a flattened ellipsoid at the bottom so the base stays flat, a few large balls for the main shape, and lots of smaller ones on top for the bumpy, cauliflower look you see on real cumulus clouds. After converting them to meshes, I scaled each one to about a metre so they'd match the old spheres, and added a little surface noise with a Displace modifier and a Clouds texture.

#### Glowing Cloud Edges

To make the clouds feel less like solid cotton, I added a rim light to `M_Dream`: a `Fresnel` node, multiplied by a new `RimStrength` parameter and the object's own colour, added on top of the existing glow. Fresnel is 1 at the edges of a surface and 0 where it faces the camera, so the outline of each cloud glows softly as if light were passing through it.

`RimStrength` is 0 by default, so nothing else in the scene changed. Only `MI_Cloud` turns it on. Because the rim uses the object's colour, it turns sky blue together with the cloud when it transforms.

#### A Glow While You Look

Last milestone, I chose not to add a crosshair, which made it hard to tell what you were about to change. Now objects answer your attention directly: while you look at something, it slowly starts to glow, and when the glow peaks it transforms.

I added a second function to the interface, `SetGazeProgress`. Every frame the character sends the object a value from 0 to 1 (the gaze time divided by the gaze duration), and when you look away it sends 0 so the glow fades out. On start, each object remembers its own glow level, and while you're looking it adds up to `GlowBoost` on top of that. Once the transformation begins, the extra glow fades back down over the same timeline.

I also raised the gaze time from one second to two. With one second the glow was over before you noticed it.

Two things were fiddly. The fish hardly glowed at all, because their material pulses and the pulse multiplies the glow, so for most of its cycle it was close to zero. Lowering `PulseAmount` and giving the fish a stronger `GlowBoost` fixed it. And reading the starting glow from the material only works if the `Get Scalar Parameter Value` node is wired into the execution chain. Otherwise Unreal skips it and quietly returns 0.

#### Floating Books (Bonus)

Instead of a separate bookshelf I used the drawer cabinet, since it was already in the room and its flat top is a natural place for books. `BP_Interactable_Drawer` is another child of `BP_Interactable_Base`, with five books on top of the cabinet made from thin boxes.

When the drawer transforms, it doesn't grow (its `ScaleMultiplier` is 1). Instead the books float up, each a bit higher than the one before (from 40 to 120 cm), turning and tilting slightly as they go. To keep the glow, the colour change and the sparkles working without rewriting them, all five books share one dynamic material, and the drawer hands that material to the parent class in place of its own. So the parent's logic just works on the books: they glow while you look, then turn from dark red to gold.

The Blueprint overrides `Interact`, checks that the drawer hasn't already transformed, calls the parent's `Interact`, and then plays a three-second timeline that moves every book from its saved starting position. At first the books barely moved. The maths nodes for the height weren't chained together, so each book rose by its index in centimetres (0 to 4 cm) instead of 40 to 120.

## Milestone 4 (17.11)

**Goal:** Give the dream a beginning and an ending.

- Make the room fade into white clouds or fog once every object has transformed ~ 2h
- Add soft ambient sound and a gentle sound when an object transforms ~ 1h
- Finish the window so it shows a view of another place ~ 2h
- Fade in from white when the game starts and fade back to white when the dream ends ~ 1h

**Bonus Tasks:**

- End the dream with a short cinematic camera shot
- Give the sofa or the armchair its own transformation

### Development Notes

#### Start and End Transitions

The game opens on a white screen that clears over three seconds, and ends by fading back to white. Both use `Start Camera Fade` on the player camera manager: the Level Blueprint fades from alpha 1 to 0 on start, and `BP_DreamDirector` fades from 0 to 1 at the end with *Hold When Finished* enabled. *Should Fade Audio* is enabled in both, so the music fades together with the image.

The end of the dream is handled by a new `EndDream` event in `BP_DreamDirector`. When the world transition timeline finishes, the director checks whether `TransformedCount` has reached `TotalInteractables` and, if so, calls `EndDream`.

#### Dissolving into Fog

`EndDream` plays a five-second timeline (`TL_DreamFog`) with an ease-in-out curve. It blends the height fog from its final dream state (`EndFogDensity`, `EndFogColor`) to `DreamFogDensity` and `DreamFogColor`, so the room gradually disappears into a dense, pale lilac fog. Two seconds later, the screen fades to white.

I chose pale lilac over pure white because it continues the pink and purple tones the world shifts into, while still reading as nearly white.

#### Sound

- **Background music:** a royalty-free psychedelic track (Floating in Space by geoffharvey) from Pixabay, in the spirit of late-60s Pink Floyd and the Beatles. It plays on a looping Ambient Sound without attenuation, so it is heard evenly across the room.
- **Transformation sound:** a short sound effect (Reverse Cymbal Sound Effect) from Pixabay, played with `Play Sound 2D` in `BP_Interactable_Base` together with the particle burst. The pitch is randomised between 0.9 and 1.1 to avoid repetition. Since all interactables inherit from the base class, none of the child Blueprints needed changes.

#### Window View

The window frame is mounted on a solid wall, so instead of an actual opening, the glass uses a material that fakes depth. `M_WindowView` is an unlit material showing a photo of a lilac sky (Pixabay). A `BumpOffset` node shifts the texture coordinates based on the viewing angle, creating a parallax effect that makes the sky appear to sit far behind the glass.

`MI_WindowView` exposes these parameters:
- `ScaleU` and `ScaleV` scale the texture separately, since the window is tall and the photo is wide
- `OffsetU` and `OffsetV` select the visible part of the photo
- `Depth` controls the strength of the parallax
- `Brightness` is kept low, because auto exposure in the darker room would otherwise wash out the unlit image

The doorway shows the plain outside, while the window shows a different place entirely.

#### Floating Armchair (Bonus)

`BP_Interactable_Armchair` is a child of `BP_Interactable_Base`. When it transforms, the chair rises 80 cm over three seconds while its colour shifts towards lilac, then keeps floating in the air, bobbing gently and rotating slowly.

**Material:** The armchair's imported material routes its colour through `MF_PhongToMetalRoughness`. I added a `Tint` parameter multiplied after this function and a `GlowStrength` parameter driving Emissive, so the parent's gaze glow and colour logic work without changes. Because the tint is multiplied, the green turns into a muted, greyish lilac rather than a pure one.

**Movement:**
- `TL_Rise` (3 s) moves the chair up from its stored start location and blends `Tint` from white to `DreamTint`.
- When the timeline finishes, `bFloating` is set and `Tick` takes over. The height is `FloatHeight + sin(FloatTime × 1.5) × BobHeight` (72 to 88 cm by default), and the chair rotates by `SpinSpeed × DeltaSeconds` around the vertical axis.
- `FloatTime` starts at zero at the top of the rise, so the switch to bobbing is smooth.
