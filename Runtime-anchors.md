Runtime anchors, or simply "anchors", are a way for objects to reference something (another GameObject or Component) which is not available in Edit mode, and thus needs to be provided at runtime. They generally cover two needs:
- Referencing a GameObject or Component which is instantiated during gameplay.
- Referencing a GameObject or Component which is in another scene.

Since they are a ScriptableObject, Runtime anchors provide an "anchor" that the designer can point to while setting up a Prefab, so that the object can find that reference later, at runtime, when it's made available.

Using this pattern (and other ScriptableObject-based references), we have Prefabs in the project that can be dropped in a scene and will work automatically, without a need for cross-object connections to be setup. These connections will be made at runtime.

![Runtime Anchors setup](https://github.com/UnityTechnologies/open-project-1/raw/main/Docs/WikiImages/RuntimeAnchors_Setup.jpg)

## Table of Contents
- [Usage example](#usage-example)
- [More info](#more-info)

### Usage example
One of the main uses of this pattern in the project revolves around the player character. Since the protagonist is spawned (and respawned) at runtime, we can't set it as a target for Cinemachine in Edit mode. To achieve this, the `CameraManager` script refers to a `TransformAnchor` ScriptableObject, knowing that at runtime, _at some point_, it's going to contain the Transform of the player character.

#### Providing the value
The value contained in this anchor is managed at runtime by the `SpawnSystem` script, which is responsible for spawning the player. This script provides the `TransformAnchor` with the correct Transform value in its `SpawnPlayer()` method:
```cs
Protagonist playerInstance = Instantiate(_playerPrefab, spawnLocation.position, spawnLocation.rotation);
_playerTransformAnchor.Provide(playerInstance.transform);
```

When a new scene is loaded and the player is destroyed, `SpawnSystem` takes care to invalidate the Transform reference contained in the anchor in its `OnDisable()`:

```cs
_playerTransformAnchor.Unset();
```

#### Using the value
On the `CameraManager` side, we call a function (`SetupProtagonistVirtualCamera()`) to setup Cinemachine with the correct target. The function is called in Start, but only if the Transform is already available (meaning the protagonist has been already spawned).

```cs
//Setup the camera target if the protagonist is already available
if(_protagonistTransformAnchor.isSet)
	SetupProtagonistVirtualCamera();
```

It's always a good idea to check the `isSet` property, to verify that the value within the anchor is not null.

The `CameraManager` also subscribes to the Runtime anchor's event, so that it's notified in case the Transform contained in it were to change. This way, whenever the player is respawned, the camera is setup again and frames the correct Transform:

```cs
_protagonistTransformAnchor.OnAnchorProvided += SetupProtagonistVirtualCamera;
```

Refer to this image for a diagram of how the Transform value is provided, and used, at runtime:
![Runtime Anchors at runtime](https://github.com/UnityTechnologies/open-project-1/raw/main/Docs/WikiImages/RuntimeAnchors_Using.jpg)

### More info
Runtime Anchors are our implementation of a similar concept from other projects with similar needs. In the below resources, they are referred to as "Runtime Sets", and they can contain multiple objects of the same type - while we generally use them for one object.
- [Ryan Hipple's original video on ScriptableObjects patterns, Unite Austin 2017](https://youtu.be/raQ3iHhE_Kk?t=2311) (specific timing for Runtime Sets linked)
- [Another video on Runtime Sets, by Matt Schell](https://www.youtube.com/watch?v=Wo2qQPqfYJs)