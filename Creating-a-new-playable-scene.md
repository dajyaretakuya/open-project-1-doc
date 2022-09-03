If you want to create a new location for the game, a good place to start is the scene _EmptyLocation, found in `/Scenes/Locations/`.

To create a location completely from scratch instead and be able to play in it, you need to provide a few fundamental Prefabs. They are all located in the folder `/Prefabs/Gameplay/`.

1. Add the **EditorInitializer** Prefab  
The script on this Prefab (EditorColdStartup component) loads the necessary "manager" scenes to make the game playable and to allow the player character to move between Locations. If this Prefab is missing, the player character will not be spawned when pressing Play.  
**Important:** This script needs to reference a ScriptableObject of type `SceneData` to work. This is what tells the loading system what scene the game is starting from. Create one (preferably in the folder `/ScriptableObjects/SceneData/`) and make sure its **Scene Type** property is set to **Location**, then reference the correct scene file in it. Finally, reference this newly created ScriptableObject into the EditorColdStartup component, in the property **This Scene SO**.

2.  Add the **CameraSystem** Prefab  
This Prefab contains the Camera (which includes the CinemachineBrain component) and the main Cinemachine virtual camera, a FreeLook, which follows the player character during gameplay. The FreeLook initially contains a dummy target, which is swapped dynamically with the player character when the game enters Play Mode.  
If present, remove the default Camera GameObject.

3. Add the **SpawnSystem** Prefab  
This Prefab takes care of spawning the player. In a normal game situation, it would spawn it in a specific position (entry point) depending on which scene the player is coming from. When testing individual scenes, it will spawn it in a default location (see children GameObjects).

4. Add some geometry to walk on  
Add a plane, or any geometry that the character can walk on. Ensure it has a collider (any), and that the default spawn location GameObject (in the SpawnSystem Prefab) is placed above it.

5. Press Play