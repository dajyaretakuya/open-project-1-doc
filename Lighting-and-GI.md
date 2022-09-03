The lighting model in Chop Chop is pretty simple, and it relies a lot on a custom shader to deliver its look. In short, we bake GI using Subtractive mode mixed lighting and then we process lightmaps within the shader, while moving objects receive a mix of realtime light and light from Light Probes.

![Shadows](https://github.com/UnityTechnologies/open-project-1/blob/main/Docs/WikiImages/Shadows.png?raw=true)

As pictured in the image above, static objects (like the tree) have baked shadows, resampled by the toon shader. These shadows never become fully black. Dynamic objects instead (like the Plant Critter) cast a very dark shadow on static objects (the ground in this case), but not on dynamic objects (other characters) or static objects lit by light probes (like the little rock, pictured).

## Table of Contents
- [Light setup](#Light-setup)
- [Objects setup](#Objects-setup)
- [Light probe setup](#Light-probe-setup)
- [Ambient lighting](#Ambient-lighting)
- [Skybox](#Skybox)
- [Baking settings](#Baking-settings)

### Light setup
The scene setup is extremely simple: we have just one directional light representing the sun and set to Mixed mode. This light has intensity 1 and colour #FFF4D6 (for full day scenes), and is set to provide Soft Shadows.

### Objects setup
All big pieces of environment geometry are set to static (Contribute GI), while all small objects (like grass, pebbles, flowers) are set to static but the Receive Global Illumination property is set to Light Probes on their MeshRenderer components.

You might notice that despite being set as static, many objects don't have the Batching Static flag on. This is because some objects (mainly nature and plants) use a shader that relies on object position to tint it randomly. Batching objects together breaks the shader since positional information is lost, so we disable this property to avoid the mesh being batched. On these objects we usually enable [GPU Instancing](https://docs.unity3d.com/Manual/GPUInstancing.html) on the material.

#### Scale in Lightmap
Since we bake at a very low lightmap resolution (see [Baking settings](#Baking-settings)), some objects with high frequency details but small size used to produce light artifacts with dark areas even in parts that were fully lit. To cope with this, we use a Scale in Lightmap factor higher than 1 on those Prefabs (usually 1.5 to 2), effectively raising the baking resolution ad-hoc just for those objects.

Examples of this are the two stools, and the ferns present on the beach scene.

**[TODO: ADD STOOLS IMAGE]**

### Light Probe setup
Since hand-placing Light Probes is a tedious setup, we have embedded a Light Probe group object into many Prefabs that are standing upright, like trees, houses, torches, bushes, grass, and more. This makes it so that when baking lights, a bunch of light probes will automatically be added to the network of probes without us having to place them one by one. The objects containing the Light Probe group is tagged as EditorOnly, so they are stripped and don't have any cost on the build.

**[TODO: ADD PROBE PREFAB PIC]**

We don't include these probes on Prefabs that are placed rotated, like rocks, since that would often put probes under the ground or into walls, creating unwanted "pitch black" probes that would produce glitches in the lighting result.

In addition to these "automated" probes, we also add a few hand-placed ones in key points of each location.

### Ambient lighting
Ambient lighting is provided using the colour of the skybox.

**[TODO: SWAP PICTURE]** [Ambient Settings](https://github.com/UnityTechnologies/open-project-1/blob/main/Docs/WikiImages/AmbientSettings.png?raw=true)

### Skybox
The skybox is a Cubemap skybox material, using custom painted cubemap textures, found in `/Art/Skybox/Painted`.

### Baking settings
For baking GI, we use Mixed lighting mode set to Subtractive. Realtime shadow colour doesn't matter, since it's forced to a different colour in the shader.

When we bake lightmaps, much of the nuance of the lightmaps is actually lost because of the toon shader approximating gradients. For this reason, many baking parameters like the number of samples don't make much of a difference.

Since from Unity 2020.3 LTS lighting settings are stored in a file, you can use the ones coming from a file called "Locations', which is found in `/Settings/LightBakingSettings/`. Please don't modify the values of this file, as it is used to bake all Locations in the game. If you want to make quick tests or have to bake non-final scenes, such as whiteboxing, use the file "LowResBakes" in the same folder.

![Baking settings](https://github.com/UnityTechnologies/open-project-1/blob/main/Docs/WikiImages/BakingSettings.png?raw=true)

If you want to create your own settings, the most important parameters are the ones as follows:
- We bake **Ambient Occlusion** and sometimes with Direct Contribution set to 2, to make it stand out a bit more.
- **Lightmap resolution** is set to very low values, like 10 or even 8 in some big scenes. When making test bakes, we go down to 2.
- Directional mode is set to **Non-Directional** because we don't need that extra set of maps, given that we don't use a PBR workflow.

## Night scenes
For night scenes (currently only a test, found at `/Scenes/WIP/NightGround.unity`) decrease the intensity of the main light to .1 before baking. Once baking is complete, bring it back to 1, so it lights dynamic objects correctly.