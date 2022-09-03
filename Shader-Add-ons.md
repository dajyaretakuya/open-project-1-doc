Besides the base [shading model](https://github.com/UnityTechnologies/open-project-1/wiki/Shading-Model) to define light/shadow areas, we have a system to add extra functionality to shader that we call "shader add-ons”. We have the base shader (`/Shaders/Toon.shadergraph`) and, on top of that, we add additional effects in the form of these add-ons.

To drive this, we make big use of Subgraphs in Shader Graph. Shader add-ons are modules that can be added - potentially with other add-ons - to the base shading model (which, by the way, is also encapsulated in a Subgraph found at `/Shaders/Subgraphs/ToonLighting.subgraph`).

This allows us to have shaders for:
- Toon
- Toon + Wind
- Toon + Wind + Masking + Tinting
- Toon + Dissolve

... and so on.

To better understand this organisation, see the diagram below and check out the shader `/Shaders/Toon_Wind.shadergraph` in the project:

![Shader Add-ons](https://github.com/UnityTechnologies/open-project-1/blob/main/Docs/WikiImages/ShaderAddons.png)
_See this image bigger [on the task card](https://open.codecks.io/unity-open-project-1/decks/61-game-elements/card/17h-shader-add-on-effects?show=fileViewer&m.url=https%3A%2F%2Fuploads.codecks.io%2Faccount-b20b3b1e-ca70-11ea-b9c6-6bde57da0873%2F2020%2FhK3iG4g0iY%2Fimage.png) in the roadmap_