Input in Chop Chop uses Unity's "new" [Input System package](https://docs.unity3d.com/Packages/com.unity.inputsystem@1.1/manual/QuickStartGuide.html). On top of that, we have built some intermediate layers to process the input coming from physical devices and handle the scenarios required by this particular game.

## Table of Contents
- [Basic setup](#basic-setup)
- [Input processing](#input-processing)
- [Action maps](#action-maps)
- [More info](#more-info)

### Basic setup
![Input Actions mapping](https://github.com/UnityTechnologies/open-project-1/blob/main/Docs/WikiImages/InputActions.png)

To read the Actions from the InputActions asset (pictured above, and which you can find in `/Settings/Input/GameInput.inputactions`), we use what is called **Generate C# Class**, which creates a C# file that contains the interfaces we implement in the scripts. This removes the need to look up for Actions using a string, and allows us to have auto-completion in code. You can find more info on this technique [in the package manual](https://docs.unity3d.com/Packages/com.unity.inputsystem@1.1/manual/ActionAssets.html#auto-generating-script-code-for-actions).

The file generated can be found under `/Scripts/Input/GameInput.cs`.

### Input processing
Once we have the C# file, we create callbacks inside a ScriptableObject called "InputReader", of which only one instance exists in the game. All GameObjects that need to access input reference this ScriptableObject, so they are all reading from the same source. The reason for this layer is that we process some of the input before making it available to scripts.

For instance, here's an extract from `InputReader.cs`. Here we first have the delegate `UnityAction`s, which other scripts hook into. When the Input System invokes the callback `OnJump` as a result of pressing or releasing the jump button, we evaluate the phase of the input ('performed' or 'canceled') and fire the appropriate `UnityAction` to create two different phases for the jump (ascending and descending) based on the same button.

```cs
// Gameplay
public event UnityAction jumpEvent = delegate { };
public event UnityAction jumpCanceledEvent = delegate { };

public void OnJump(InputAction.CallbackContext context)
{
	if (context.phase == InputActionPhase.Performed)
		jumpEvent.Invoke();

	if (context.phase == InputActionPhase.Canceled)
		jumpCanceledEvent.Invoke();
}
```

### Action Maps
We use Action Maps to differentiate between different phases of gameplay. We have so far:
- Gameplay: This is active whenever you have control of the Pig Chef character, and it includes moving around, jumping, fighting, and interacting with cooking pot, NPCs, plus pausing the game.
- Menus: Active in all menus (main menu, settings, pause menu, inventory) and includes mostly moving the selection around, confirming, canceling, and unpausing (only works in the pause menu).
- Dialogues: Similar to Menus, this includes only moving the selection around and confirming. It's used in dialogues with multiple choices, to decide which answer to give. Cutscenes, having no multiple choice dialogues, don't fall into the Dialogues Action Map even though a character might be talking.

Switching Action Maps happens in the `InputReader.cs` class (see above), through functions like `EnableDialogueInput()` or `DisableAllInput()` (useful for cutscenes).

### More info
- You can find a high-level overview of the implementation above in [Devlog 3: Handling complex input](https://youtu.be/u1Zel20rwOk), available on YouTube.