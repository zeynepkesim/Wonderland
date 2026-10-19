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

#### Scene and atmosphere

I started from Unreal's First Person template (Blueprint) and built my own level, `L_Wonderland_Main`. The scene is a room open to the sky, with a doorway leading out into the clouds. A low sun shines in through the doorway, and volumetric fog turns its light into visible shafts. Sky atmosphere, volumetric clouds and a post-process volume (bloom, vignette) complete the soft, dream-like look.

#### Environment

Most of the environment is still a blockout made from basic shapes: the walls, an armchair, a window, a mirror, and clouds piled around and inside the room. The goldfish are an exception. I modelled a simple low-poly goldfish in Blender (body, tail and dorsal fin), exported it as FBX and imported it as a static mesh. Small goldfish float around the room, and one giant fish hangs outside the doorway to play with scale.

#### Materials

Everything uses a single master material, `M_Dream`. It blends between two colours (`ColorA` and `ColorB`) using a `Transform` parameter from 0 to 1, and also has parameters for glow, pulsing, roughness and metallic. Each object type has its own material instance with its own palette: turquoise walls, moss-green floor and armchair, orange fish, white clouds, a glowing window and a silver mirror. The second colour of each instance is the colour the object turns into when it transforms.

#### Movement and camera

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

#### Looking instead of pressing

The E key is gone. Now you just look at something for about a second and it changes.

Every frame, the character casts a sphere trace (40 cm radius, up to 5 metres) from the centre of the camera. I switched from a line trace because a thin line kept slipping off objects, which made it almost impossible to keep looking at one thing for a whole second. If the trace keeps hitting the same interactable object, a timer counts up. Look away or at something else and it resets. Once it reaches `GazeDuration` (1 second by default), the object gets the same `Interact` message as before. So the transformation itself didn't change at all, only what triggers it.

I also made transformations one-way. Toggling back and forth made sense with a key, but with gaze an object would flip back if you just kept staring at it, which felt confusing. Now once something changes, it stays changed.

#### The world slowly changes

`BP_DreamDirector` is an invisible actor that keeps track of how deep into the dream you are. When the game starts, it counts every actor that implements `BPI_Interactable` and saves the starting fog values. Every time an object transforms, it lets the director know. The director works out how many objects have changed so far (say 3 out of 7) and moves the whole world towards a "deeper" look over three seconds.

That deeper look comes from two things. The height fog gets thicker and shifts from pale turquoise to pink and purple, and a second post-process volume (`PPV_DeepDream`) fades in, changing the colour temperature, tint, bloom and vignette. Everything goes through one function, `ApplyWorldState`, which takes a single value from 0 to 1. At 0 the room looks like it did in Milestone 1, at 1 every object has transformed.

#### Swimming goldfish

The goldfish now swim in slow circles around the spot where they were placed, bobbing gently up and down and always facing the way they're going. Each fish starts at a random point on its circle, so they don't all move in sync. Radius, speed and bob height can be set per fish, so the giant one outside can move slower and in a wider circle. It still works with gaze, so you can follow a fish with your eyes until it turns gold.

#### New armchair, window and mirror

The block armchair is now *GreenChair_01* from Poly Haven. Its green fabric fit the palette so well that I kept the original textures.

For the window and the mirror I used *Ornate Mirror 01*, also from Poly Haven. The model came with a single material for both the frame and the glass, so I opened it in Blender and moved the glass faces into their own material slot. Now the same gilded frame shows up twice on the wall: in the window the glass glows (`MI_Window`), and in the mirror it's a smooth silver surface that actually reflects the room (`MI_Mirror`).

#### Sparkles

When an object starts to transform, a small burst of glowing sparks appears around it. It's a Niagara system (`NS_TransformBurst`) made from one of the burst templates, with lower gravity and slower particles so they float instead of falling. It's attached to the object's mesh and scales with it, so it follows a swimming fish and gets bigger on the giant one.