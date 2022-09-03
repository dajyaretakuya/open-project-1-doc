This page is designed to give you a quick overview of the game architecture.

## Table of Contents
- [Scenes](#Scenes)
- [Managers](#Managers-and-systems)
- [State machine](#State-machine)
- [Event system](#Event-system)
- [Game data](#Game-data)
- [Addressables and AssetBundles](#Addressables-and-AssetBundles)

### Scenes
We use additive scene loading in the game to allow having multiple scenes loaded at the same time. Scene loading and unloading is all done by a `SceneLoader` script living in a scene called PersistentManagers (see below). The scene flow is as follows:

- The game always begins in a Unity scene called Initialization. This scene is almost empty, except for a script that loads the scene PersistentManagers and then requests the loading of MainMenu. At this point the scene Initialization unloads itself.
- PersistentManagers is a scene that is never unloaded throughout the whole game, and houses the managers which perform across-scene operations like playing audio or loading new scenes. If you ever start the game from any other Unity scene, there is a little editor-only hack (a Prefab called EditorInizialization) that will load PersistentManagers and other important scenes, so that these scripts are present.
- Right after the Initialization scene has given control to PersistentManagers, MainMenu is the first real scene loaded. This houses all the main menu UI, settings, and allows to begin gameplay.
- When gameplay begins the loading system will load/unload what we call **Locations**. In addition to these, a permanent Gameplay scene is always loaded for as long as the player is in gameplay. This contains gameplay UI and other managers that only have a reason to be in Location scenes.
- If the player returns to the main menu, MainMenu is loaded again and all the Location scenes are unloaded, as well as Gameplay.

![Scene architecture diagram](https://github.com/UnityTechnologies/open-project-1/blob/main/Docs/WikiImages/SceneArchitecture_diagram.jpg)
(this diagram is [also available on Miro](https://miro.com/app/board/o9J_khLZ3kQ=/?moveToWidget=3074457353275877659&cot=14) for bigger text)

On top of the native Unity scenes, we have a data layer made of ScriptableObjects called `SceneDataSO`. These bits of data are passed around and contain the reference to the Unity Scene file, in addition to some useful data like a descriptive Location name, and more. You can find them in the folder `/ScriptableObjects/SceneData/`.

To create a new Location scene, check out [the related page](https://github.com/UnityTechnologies/open-project-1/wiki/Creating-a-new-playable-scene).

### Managers and systems
Scripts that end with -Manager and -System are MonoBehaviours which take care of entire parts of the game. Think of them like Singletons, except we don't use the Singleton pattern but an event-driven architecture to have these scripts listen to specific in-game events and react  to them.

### State machine
Characters in the game are powered via a community-made **Finite State Machine** (FSM) solution. This State Machine is first composed with a series of interconnected ScriptableObjects (found in `ScriptableObjects/StateMachine/`) which act as the editor-time data. Then, once in Play Mode, is powered by a MonoBehaviour present on the character itself, which first kicks off the first _State_, and from then it plays back _Actions_ (which can run upon entering or exiting a state, or per-frame) and evaluates _Conditions_ to determine which state to move to next. Actions and Conditions can access MonoBehaviour components on the host GameObject, to drive its parts (good examples are the `Animator`, the `CharacterController`, or particle systems). 

Find out more in [the specific State Machine page](https://github.com/UnityTechnologies/open-project-1/wiki/State-Machine).

### Event system
The event system is the backbone of the game's architecture. We use ScriptableObjects that we call **Event Channels** to connect GameObjects between themselves, with Managers, and across scenes.

Get familiar with them by checking out [the Event System page](https://github.com/UnityTechnologies/open-project-1/wiki/Event-system).

### Game data
We rely on ScriptableObjects to drive the data in the game. Dialogues, quests, character names, events, state machine actions and conditions, and much more, are all stored in the folder `/ScriptableObjects/`. All of the data that is strings is localised using Unity's [Localisation package](https://docs.unity3d.com/Packages/com.unity.localization@0.9/manual/index.html).

### Addressables and AssetBundles
We use AssetBundles to package scenes, Prefabs, and everything in the game into chunks of data that we can load when necessary. To achieve this, we use the [Addressable Assets package](https://docs.unity3d.com/Packages/com.unity.addressables@1.17/manual/index.html), which builds an abstraction layer over AssetBundles and makes the packaging easier.

You can read more about packaging assets on the page dedicated to [building the game](https://github.com/UnityTechnologies/open-project-1/wiki/Building-the-game).