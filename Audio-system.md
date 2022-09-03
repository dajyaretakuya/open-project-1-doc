Audio in Chop Chop is played with Unity's native AudioSource and AudioListener components without any external middleware, and we have built a layer on top to help manage audio requests from different entities in the game.

## Table of contents
- [Audio Manager](#Audio-Manager)
- [Sound Emitters](#Sound-Emitters)
- [Audio Cue ScriptableObjects](#Audio-Cue-ScriptableObjects)
- [Audio Cue components](#Audio-Cue-components)
- [Audio Configs](#Audio-Configs)

### Audio Manager
The `AudioManager` is a script sitting in the PersistentManagers scene, which is a scene loaded at all times (see [here](https://github.com/UnityTechnologies/open-project-1/wiki/Game-architecture-overview#Scenes) for more info). Being in a separate scene means it can manage audio and potentially cross-fade music across scenes without interruption.

At the beginning of the game, the Audio Manager creates a pool of `SoundEmitter` objects, each one with its own AudioSource. It then listens on two [event channels](https://github.com/UnityTechnologies/open-project-1/wiki/Event-system), PlaySFX and PlayMusic, for requests coming from other objects. Once it receives a request it will pass it onto one of the `SoundEmitter`s in the pool and play music or an SFX.

Audio requests travelling on an `AudioCueEventChannel` bear three main parameters: the `AudioCueSO` they want to play, the `AudioConfig` which determines "how" the sound is played, and the position for 3D sounds.

When playing an SFX, the Audio Manager will rely on a collection of emitters of type `SoundEmitterVault`, which, through an `AudioCueKey` identifier, finds which of the objects in the pool is busy with playing which sound(s). We have a structure for playing multiple SFXs at the same time as if they were one, allowing designers to create composite sounds from within Unity (see `AudioCueSO` [below](#Audio-Cue-ScriptableObjects)).

When playing music, it will always just go to the same `SoundEmitter` and ask it to play, fade in or out, or stop. We don't currently have support for cross-fading music.

![Audio Manager script](https://github.com/UnityTechnologies/open-project-1/blob/main/Docs/WikiImages/AudioManager.png)

### Sound Emitters
The `SoundEmitter` scripts receive direct orders from the `AudioManager` to play sounds. From there, they take control of a sound and they notify the `AudioManager` when they are done (for non-looping sounds), which puts them back into the available pool. Example methods are `PlayAudioClip`, `FadeMusicIn`, `FadeMusicOut`, `Pause`, `Resume`, `Stop`, `Finish`, and more.

A `SoundEmitter` takes care of only one sound, so for composite SFX multiple `SoundEmitter` instances receive the request, each one spinning up its own `AudioSource`. In fact, some of them might be "done" before others, and go back in the pool before the whole SFX is over.

![Sound Emitters pool](https://github.com/UnityTechnologies/open-project-1/blob/main/Docs/WikiImages/SoundEmitters_Pool.png)

### Audio Cue ScriptableObjects
An `AudioCueSO` is a ScriptableObject that acts as an index of individual AudioClips that make up a complex sound. If multiple AudioClips belong to the same group, they can be played in random order or with the same order every time.

![Audio Cue SO to play a composite sound](https://github.com/UnityTechnologies/open-project-1/blob/main/Docs/WikiImages/AudioCue_CompositeSFX.png)
_The sound for the cooking pot is a looping sound, made up of two different SFXs (fire and boiling water) of different lenghts_

![Audio Cue SO to play a random sound from a list](https://github.com/UnityTechnologies/open-project-1/blob/main/Docs/WikiImages/AudioCue_RandomSFX.png)
_The sound for the player receiving a hit is randomly selected from 3 different clips each time, and it doesn't loop_

### Audio Cue components
The `AudioCue` class is a component added to Prefabs in the scene, to create positional 3D sounds that can be placed and configured by a designer, and which will usually play as soon as the scene starts. An example of this are the torches, the cooking pot, the waterfall, and other environmental sounds.

![The Audio Cue component](https://github.com/UnityTechnologies/open-project-1/blob/main/Docs/WikiImages/AudioCue_Component.png)

Audio Cues can also be played by another script, but this is rarely used in the game.