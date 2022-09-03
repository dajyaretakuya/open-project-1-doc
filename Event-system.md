![Event System Diagram](https://github.com/UnityTechnologies/open-project-1/blob/main/Docs/WikiImages/EventSystem_diagram.png)

This ScriptableObject-based event system is our solution to create a solid game architecture and make objects communicate with each other, at the same time avoiding the use of the Singleton pattern.

The reasons why we want to avoid using Singletons are multiple. They create rigid connections between different systems in the game, and that makes it so they can't exist separately and they will always depend on each other (i.e. _SystemA always requires SystemB in the scene to work_, and so on). This is of course hard to maintain and reuse, and makes testing individual systems harder without testing the whole game.

**How it works**  
At the base of the system we have a series of ScriptableObjects that we call "Event Channels". They act as channels (like a radio) on which scripts can "broadcast" events. Other scripts can in turn listen to a specific channel, and they would pick that event up, and implement a reaction (callback) to it. The graph at the top of this page visualises this structure.

Both the script that fires the event and the event listener are Monobehaviours. Since we are using the ScriptableObject Event Channel (which is an asset) to connect the 2 systems, those Monobehaviours can live in 2 different scenes in a completely independent way.

For instance, an event can be raised when pressing a button, and broadcasted on a "Button_X_Pressed" Event Channel ScriptableObject. On the other hand, we can have one or more objects listening to this event and react with different functionality: one of them spawns some particles, one plays a sound, one starts a cutscene.

## Table of Contents
- [Events channels in the project](#event-channels-in-the-project)
- Using the system
     - [Creating an Event Channel ScriptableObject](#creating-an-Event-Channel-ScriptableObject)
     - [Setting up an Event Listener](#Setting-up-an-Event-listener)
     - [Have a Script Broadcast on an Event Channel](#have-a-script-broadcast-on-an-event-channel)
- [Add a new type of Event Channel](#adding-a-new-type-of-Event-Channel)
- [Add a new type of Listener - Example](#adding-a-new-type-of-Listener)
- [More info](#more-info)

### Event Channels in the project
Events can be raised with or without parameters. Here is a few examples of event channels we have in the project:
* **Void Events** are events that are raised with no parameters. A good example of this is when we raise an event to quit the game. 
* **Int Events** are events that are raised with an `int `parameter. We can use them when we unlock an achievement, for instance, by passing the achievement ID.
* **Load Events** are events that are raised with 2 parameters, an array of `GameScene` which contain the data about the scenes we want to load, and a `bool` parameter to specify if we want to display a loading screen.

More types of channels will be added in time. Check the `/Scripts/Events/ScriptableObjects/` folder to find all the scripts that define Event Channels ScriptableObjects.

### Creating an Event Channel ScriptableObject
To create an event channel, right-click in the Project window, then under "Game Event" choose the type of event channel that best fits your case, and give it a descriptive name. The channel is ready to be used.

### Have a script broadcast on an Event Channel
To have any piece of code raise an event, you only need one line of code, provided you have a reference to a SO that acts as the channel. For instance, to have an object fire an event when another object enter its collider trigger:

```cs
public VoidEventChannelSO OnTriggerEnterEventChannel;

private void OnTriggerEnter(Collider other)
{
	OnTriggerEnterEventChannel.RaiseEvent();
}
```

### Setting up an Event Listener
There are many ways to listen for events on an Event Channel. The object that listens could be an independent MonoBehaviour script that does just that, or the listening part could be just included in another script.

In the project, an example of an integrated listener is the `LocationLoader` script. In this case, the listener is included in the script, meaning that the script acts as container of the scene loading methods but also as a listener. This is particularly useful if we are not using a specific listener more than once in the project. (it's in `Scripts/SceneManagement/LocationLoader.cs`)

Otherwise, you can find an example of a generic listener under `Scripts/Events/VoidEventsListener.cs`. This script can be added on any GameObject and listens on a `VoidEventChannelSO`, and it has a regular UnityEvent that can be wired to any behaviour in the game as a response to the event.


### Adding a new type of Event Channel
You can add a new type of Event Channel if you need a specific number/type of parameters you want to pass via the `Raise` method. This is an example of the `IntEventChannelSO`:

```cs
[CreateAssetMenu(menuName = "Events/Int Event Channel")]
public class IntEventChannelSO : ScriptableObject
{
	public UnityAction<int> OnEventRaised;
	public void RaiseEvent(int value)
	{
		OnEventRaised.Invoke(value);
	}
}
```
Replace the `int` variable with the type you need. You can also add more than one parameter.

### Adding a new type of Listener
You can add a new type of Listener depending on how many parameters were used to raise the event. This is an example of the Int Event listener:

```cs
[System.Serializable]
public class IntEvent : UnityEvent<int>
{
}

public class IntEventListener : MonoBehaviour
{
	public IntEventChannelSO IntGameEvent;
	public IntEvent OnEventRaised;

	private void OnEnable()
	{
		//Check if the event exists to avoid errors
		if (IntGameEvent == null)
		{
			return;
		}
		IntGameEvent.eventRaised += Respond;
	}

	private void OnDisable()
	{
		if (IntGameEvent == null)
		{
			return;
		}
		IntGameEvent.eventRaised -= Respond;
	}

	public void Respond(int value)
	{
		if (OnEventRaised == null)
		{
			return;
		}
		OnEventRaised.Invoke(value);
	}
}
```

### More info
- The [2nd Devlog Video](https://www.youtube.com/watch?v=WLDgtRNK2VE) is a quick recap of how we used ScriptableObjects to drive event channels.
- We initially introduced the Event system in [Episode 2 of the Livestream](https://youtu.be/ukE73ifSrTM?t=2275) (37:55). Keep in mind that some details of the implementation might have changed since then. This wiki is more updated.
- If you would like to know more about multiple arguments version of UnityEvent, you can see [examples in the documentation](https://docs.unity3d.com/ScriptReference/Events.UnityEvent.html) using up to 4 parameters.