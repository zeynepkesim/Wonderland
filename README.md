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

- Unreal Engine 5 (Blueprints)
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