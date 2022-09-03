We have created a custom toon shader using ShaderGraph which we use for both characters and environments, to create a very specific look for the game to match our concept art. You can find the base shader and all its variations in the folder `/Shaders`.

This article helps you orient yourself in the different ShaderGraphs and Subgraphs, and is mostly referring to the base shader, called `Toon.shadergraph`. For additional effects such as wind, dissolve, tinting; refer to the [Shader Add-ons](https://github.com/UnityTechnologies/open-project-1/wiki/Shader-Add-ons) page.

## Table of Contents
- [Shading model](#Shading-model)
    - [Subgraphs](#subgraphs)
    - [Shader keywords](#shader-keywords)
    - [Custom HLSL](#custom-hlsl)
- [Using the shader in other projects](#using-the-shader-in-other-projects)
- [More info](#more-info)    

### Shading model

The shader takes care of both dynamic (**1**) and static geometry (**2**) in a slightly different way. You can find this model inside a Subgraph called `ToonShading.subgraph`. This Subgraph is responsible for transforming the scene's lighting information into the final black/white toon shading, which is then multiplied by the colours to obtain the final pixels.

![Shading Style](https://github.com/UnityTechnologies/open-project-1/blob/main/Docs/WikiImages/ShadingStyle.jpg)

**1. Dynamic geometry** (the player, NPCs, enemies, and all usable tools) are shaded with two colours: one is full light, and the other shadow. The shadow the cast on static geometry is almost fully black. They receive an additional pass of light due to Light Probes. This pass is overlaid on top and just adjusts the brightness of the existing toon shading (i.e. it doesn't produce additional toon banding).

**2. Static geometry** (scenery, buildings, nature, ground, etc.) has colours that are more gentle and not so contrasted. The toon shading is 3 stripes, and they are not as hardly defined as the ones for the character. This geometry needs GI to be baked.  The shadows they cast are also paler, and made of 3 bands depending on their intensity (they are not dynamic shadows, but part of the lightmap).

#### Subgraphs
Inside the `ToonShading.subgraph`, you'll find more Subgraphs such as `ToonLightingModel.subgraph`, which is responsible for the main directional ligth, the lightmaps, and the Light Probes, or `AdditionalLights.subgraph` which does the same thing, in a simplified way, for additional point lights (they don't cast shadows). Both of these light contributions are composited on each other to give the final lighting.

#### Shader keywords
Notice that the lighting subgraphs rely on a variety of shader keywords, such as:
- `LIGHTMAP_ON` (Multi Compile - Global)
- `_MAIN_LIGHT_SHADOWS` (Shader Feature - Global)
- `_MAIN_LIGHT_SHADOWS_CASCADE` (Shader Feature - Global)
- `_SHADOWS_SOFT` (Shader Feature - Global)
- `_ADDITIONAL_LIGHT_SHADOWS` (Shader Feature - Global)

#### Custom HLSL
The main light and additional lights information is gathered using some custom HLSL, which is embedded in the graph using [Custom Function nodes](https://docs.unity3d.com/Packages/com.unity.shadergraph@10.2/manual/Custom-Function-Node.html). You can find the HLSL code in the folder `/Shaders/CustomHLSL`.

### Using the shader in other projects
To extract the shader and use it in another project, there are a couple of things that are required.
First, all the necessary files are in the `/Shaders/` folder. Don't forget to take the SubGraphs: `ToonShading.subgraph`, `ToonLightingModel.subgraph`, and more, are all necessary to make the base `Toon` shader work. And the custom HLSL.

Then, there are a couple of options that need to be tweaked in the Universal Render Pipeline asset:
- Shadow cascades need to be enabled. Either 2 or 4. With no shadow cascades, the custom HLSL will fail and return magenta. You can probably tweak the code to work with no cascades, but it's up to you.

Keep in mind that here there are some renderer features you might not need in your project, like the ones regarding character occlusion (_OccludingObjects_, _CharacterObstructed_) and the water (_Water_).

### More info

- In earlier versions of the game, dynamic characters used to have an outline. We removed it [in this commit](https://github.com/UnityTechnologies/open-project-1/commit/067988863a7c675894580ae1e6cb26c56fe9be53), so if you're interested in the effect, you can fetch it from previous commits.
- To know more about the art style of the game, read the [Art Contribution Guidelines document](https://docs.google.com/document/d/18zqe31J8EipTiEBZuwzLyG3jH7-5teAOViLEio4uko8/edit#).
- Check out the initial introduction of the Toon Shading in [Episode 1 of the Livestream](https://youtu.be/O4N4s6BKNH0?t=2877) (47:57). Keep in mind that some details of the implementation might have changed since then.
- The video [Devlog 1: How we built the Toon Shading](https://www.youtube.com/watch?v=GGTTHOpUQDE) is a great and visual way to learn some of the above. However, keep in mind that the shading model has evolved since that video, so always use the version of the shader in the project as a reference to build something new.