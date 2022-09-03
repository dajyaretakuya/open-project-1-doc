We have recurring elements in all location scenes, divided into categories by empty separator GameObjects.

![Scene Hierarchy](https://github.com/UnityTechnologies/open-project-1/raw/main/Docs/WikiImages/Location_SceneHierarchy.png)

## Table of Contents
- [Manager prefabs](#Manager-prefabs)
- [Lighting](#Lighting)
- [Gameplay logic](#Gameplay-logic)
    - [Entrances and exits](#Entrances-and-exits)
- [Environment](#Environment)
    - [Scene chunks](#Scene-chunks)

### Manager prefabs
In this category we have some prefabs that act as global managers, who are generally necessary to play the game, and only exist in one copy. Many of these managers are waiting for an event (see [Event System](https://github.com/UnityTechnologies/open-project-1/wiki/Event-system)) called OnSceneReady to start. This event is fired by the SceneLoader script present in the scene PersistentManagers, once a location is fully loaded.

**Editor initialiser**  
A "hack" script that takes care of initialising the scene and the game when pressing Play from the editor (something that we call "cold startup"), as if the player came through the main menu in a normal gameplay flow. This script is totally useless in the build, and in fact it is stripped from it thanks to being tagged as EditorOnly.

**Spawn system**  
Takes care of spawning the player when the event OnSceneReady is fired. It uses the last path taken in the `PathStorageSO` Scriptable Object to determine in which of the many location entrances to spawn the player Prefab (see [Scene entrance and exit](#Scene-entrance-and-exit) below).

The Spawn system Prefab has a child object which represents the default location to spawn from. This location is selected if the developer presses Play in the editor, if a location is loaded from the main menu, or if for any reason the `PathStorageSO` data is missing. When testing often, you can reposition this object to save yourself some time so whenever entering Play mode the player is spawn exactly where you want.

**Camera system**  
Contains the Camera GameObject, and also the Cinemachine FreeLook Virtual Camera (VCam) that acts as the main gameplay VCam. Since this VCam needs to frame the player Prefab but this is not instantiated at the beginning of the game, the Camera System script listens to an event called PlayerInstantiated which notifies it and also carries information on the Transform to frame.

It also acts as a proxy to propagate shake events to the child object ImpulseSource, receiving the notification on the CameraShake event channel coming mainly from combat actions in the [State machine](https://github.com/UnityTechnologies/open-project-1/wiki/State-machine) of the characters receiving damage.

**Music player**  
Fires an event on an [AudioCueEventChannel](https://github.com/UnityTechnologies/open-project-1/wiki/Event-system) to notify the AudioManager script (located in the PersistentManagers scene, which is loaded at the same time at runtime) to play the music associated to the current location.

### Lighting
The lighting section contains the main directional light (a Prefab that changes according to the time of the day in each scene), the light probes, and in some special cases other lights (like for the Cave location).

For light probes, the Light Probe group you find here contains just the unique light probes belonging to each location. However, many more light probes are embedded into Prefabs and merged with the unique ones. Find more details on the [Lighting and GI page](https://github.com/UnityTechnologies/open-project-1/wiki/Lighting-and-GI#Light-probe-setup).

Other lights might be present in the scene, such as for instance the ones coming from torches or cooking pots. Find them all using the [Light Explorer window](https://docs.unity3d.com/Manual/LightingExplorer.html).

### Gameplay logic
This group contains all objects that are somehow "interactive" for the player. Among them, a Prefab called Interactables_[location], which includes all NPCs, enemies, cooking pots, and other interactive elements.

Cutscenes might also be present here (not in all locations) and they are usually "self-sufficient", meaning that all of the objects referenced by the Timeline are included in the Prefab.

#### Entrances and exits
A location can have multiple entrances and exits and they are supposed to be connecting physical locations, so we have setup a mechanism to determine - once a new scene is loaded - where the player should be entering it from. This relies on a Scriptable Object of type `PathStorageSO`, referenced by multiple scripts in the game. This unique Scriptable Object contains a reference to a `PathSO` Scriptable Object, which identifies a virtual path connecting two locations. Paths have no direction, that's why they are used for both locations connected. If locations in the island can be imagined as nodes on a network, think of paths as virtual nodes that have no physical representation but connect two location nodes.

![Entrance and Exit in the Beach scene](https://github.com/UnityTechnologies/open-project-1/raw/main/Docs/WikiImages/Entrance_Exit.png)

**Location exit**  
These GameObjects have a trigger collider to detect the player which, OnTriggerEnter, fires an event on the LoadLocation event channel to notify the SceneLoader to load a new scene. At the same time, the exit writes into the `PathStorageSO` which `PathSO` to consider the last taken path.

**Location entrance**  
When the scene is loaded, the Spawn system finds all entrance objects in the scene and puts them in a list. When the player needs to be spawned, the Spawn system checks which entrance script references the same `PathSO` that is referenced in the `PathStorageSO` (ideally set by the previous location's exit). This means that the player took an exit in the previous scene that was supposed to take to _this entrance_, and so it spawns the player there.

### Environment
The last group contains all objects that are static and non-interactable. This usually includes the sea Prefab with the sea floor; one or more landmass Prefabs that act as the walkable base of a location; vegetation, rocks, tiny objects, and man-made structures.  

To break up the world of Chop Chop, we chose not to use Additive scene loading. Instead, we always have one scene open at a time, and we make big use of macro-Prefabs as containers for individual Prefabs, to create reusable blocks that repeat in multiple scenes in order to help with the authoring.

When modifying scenes, it's important to do so by entering Prefab mode, in order not to leave overrides in the scenes that would result in a mismatch between locations.

**Landmasses**  
These meshes are usually planes modelled with ProBuilder. They use a material using a specific shader that allows to paint vertex colour to mix 4 ground textures (grass, cobblestone, sand and dirt). Check the Toon_Ground_VertexColour shader in `/Shaders/` to see how it works.

**Note:** it is important to modify these meshes (either their geometry or the vertex colours) inside the Prefab they belong to. If you do it in the scene, the modifications will be added as an override to the Prefab, and won't propagate to other scenes.

![Landmass Overrides dropdown](https://github.com/UnityTechnologies/open-project-1/blob/main/Docs/WikiImages/Landmass_OverridesDropdown.png)

#### Scene chunks
"Scene chunks" is a name we have given to mega-Prefabs that represent whole sections of a location, usually bunches of rocks that act as visual dividers between locations to create separation and allow us to hide other scenes, in order not to create an endless visual field. The peculiarity of scene chunks is that often there are two versions of the same Prefab, one with less details (let's call them _blockout_) and one with more (the _closeups_).

You could think of them almost as two levels of an LOD, except that, since the game is not open world, we never get to have both level of detail in the same scene since we don't move away from the closeup ones, and we don't move close enough to the blockout ones. With respect to a pure LOD solution (like the one built-in in Unity) this has a couple of advantages: first, we're not holding multiple meshes in memory, and second, we don't bake both versions in each scene so we save quite some space on lightmaps too.

Each location is usually composed of a bunch of closeup scene chunks (identified by the "-CL" termination in the name, which stands for "closeup") that are close to the player and sometimes provide walking ground. In the background, you will see a bunch of blockout chunks for scenes that are out of reach.  
As the player moves to another location, that new location will be composed of closeup chunks for that location, and again blockout chunks of other locations, and so on.

**Example:** the first scene in the game, Beach, contains all closeup chunks for the beach itself and for the close Prefabs up the hill, but only blockout Prefabs for the sides of the hill that are barely visible to the player.

![Scene chunks in the Beach location](https://github.com/UnityTechnologies/open-project-1/blob/main/Docs/WikiImages/SceneChunks.png)

Most of the distant geometry from faraway scenes is captured in the skybox, which is baked by a Reflection Probe in a special scene called IslandMaster, located in `/Scenes/WIP`.