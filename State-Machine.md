![State Machine diagram](https://github.com/UnityTechnologies/open-project-1/blob/main/Docs/WikiImages/StateMachine_diagram.jpg)
([bigger version on Miro](https://miro.com/app/board/o9J_khLZ3kQ=/?moveToWidget=3074457350985450798&cot=14))

A state machine is an object that can be in one of multiple **states**. Each of these **states** performs a specific set of **actions**, and can **transition** to different **states** based on predefined **conditions**.

In Chop Chop, we implemented a custom state machine solution based on ScriptableObjects, and we use it mainly to power characters (including the main protagonist). See below for details on the implementation.

## Table Of Contents
- [Two-Parts implementation](#two-parts-implementation)
	- [ScriptableObjects](#scriptableobjects)
	- [Runtime](#runtime)
- [How-tos](#how-tos)
	- [Creating a State](#creating-a-state)
	- [Creating Action and Conditions](#creating-actions-and-conditions)
	- [Creating a Transition Table](#creating-a-transition-table)
- [The Transition Table Editor window](#the-transition-table-editor-window)

### Two-Parts Implementation
This implementation is divided into two parts: the configuration, which happens at edit time and is based on **ScriptableObjects**, and the **runtime** part.

#### ScriptableObjects
The ScriptableObject part of the implementation is a designer-friendly, modular way of building state machines. The ScriptableObjects are used to initialise the state machine with data, which means that by creating more instances of the same type you can give different values to them and create quantitative variations of the same behaviour.

- **StateSO**: A `ScriptableObject` that can hold a reference to one or more actions (`StateActionSO`). 
- **StateActionSO**: A `ScriptableObject` that wraps a **StateAction**.
- **StateConditionSO**: A `ScriptableObject` that wraps a **Condition**.
- **TransitionTableSO**: A `ScriptableObject` that defines the **Transitions** of a state machine. This table "recaps" the structure of the state machine, and in fact we use it to display a custom editor window which allows you to have a complete overview of the state machine and all of its states, actions, transitions and conditions.

As an example, you can find the ScriptableObjects for the main character's state machine in the folder `/ScriptableObjects/StateMachine/Protagonist/`.

#### Runtime
The runtime part of the implementation has its name from being the one that contains the code that gets executed when in Play mode. Runtime objects are instances of plain C# classes, created from a corresponding `ScriptableObject` (see above).

- **State**: A class that contains multiple instances of `Action` and `Transition`. The `State` is responsible for routing the callbacks from the `StateMachine` to each `Action` and `Transition`. This class is only used internally.
- **StateAction**: A class that represents an action to perform. This is the base class all custom actions must inherit from.
- **StateTransition**: A class that contains a target `State`, and a set of `Condition` that must be evaluated to transition to it. Each `State` holds its own `Transitions` list. This class is only used internally.
- **Condition**: A class that represents a conditional statement. This is the base class all custom conditions must inherit from.
- **StateCondition**: A struct that holds a `Condition` and the expected result of the condition. This allows for reusability for when a `Condition` should sometimes evaluate to `true`, and sometimes to `false`. This struct is only used internally.
- **StateMachine**: The only `MonoBehaviour`, and the one that should be attatched to any `GameObject` that wants to make use of this implementation. It takes in a `TransitionTableSO`, and, during its `Awake` call, each `ActionSO`, `ConditionSO`, and `StateSO` in it is resolved into its runtime counterpart, and the `Transitions` lists are generated. In `Update`, `Actions` of the current `State` are performed and then `Conditions` of every `Transition` of the current `State` are checked. The first `Transition` that evaluates to `true` gets triggered.

### How-Tos
#### Creating a State
To create a new `StateSO`, simply open the Assets menu and navigate to Create > State Machines > State. Name your new State and then you can start adding actions to it. Note that the order in which the actions are displayed corresponds to the order in which they are performed; the custom editor allows for easy sorting so you can organize them accordingly. 
#### Creating Actions and Conditions
To create a new `Action` or `Condition` script, in the Assets menu go to Create > State Machines > Action/Condition Script. This will create a new script from a template that will contain both the **SO** class and the **runtime** class. You can modify both classes to your needs. Remember that after creating the script, you need to create at least one instance of the SO to use it in your state machine. The template already adds the Asset menu item for you, under Create > State Machines > Actions/Conditions > Your New Script.
#### Creating a Transition Table
You can create a new Transition Table by opening the Asset menu > Create > State Machines > Transition Table.

From there, you can start adding transitions by using the **Add transition** button. Each transition needs a 'From State' and a 'To State'. 'From State' refers to the state the state machine should be in so that it can transition to the 'To State'. After setting them, you also need to add the conditions that must be met to make the transition happen. You can chain conditions with the And/Or option, and set whether the condition should be `true` or `false`, allowing for multiple scenarios in which the transition should take place. Once everything is set up, click on **Add transition** once again to confirm your changes.

The transition table is organized by 'From States'. This means that you'll see expandable groups with the names of the 'From States' as their headers, and expanding one will list all of the 'To States' it can transition to, along with their conditions. Transitions that take priority within a 'From State' should be sorted first, as they are checked sequentially and the first one that is met will trigger the transition. You can sort them with the up and down arrow buttons by the 'To State'. You can also remove a transition with the minus (-) button. Removing all the 'To State's from a 'From State' will remove the 'From State' from the table. You can also sort the 'From States'. However, only the first one is important to set up correctly as it will be the initial state of your state machine; subsequent ordering is purely preferential. And if you need to edit a State in your Transitions Table, besides finding the asset in the project window, you can simply click on the tools (![SceneViewTools](https://raw.githubusercontent.com/halak/unity-editor-icons/master/icons/small/SceneViewTools.png)) button by the name of the State. This will display the State editor within the Table editor, allowing you to easily navigate back and forth between them.

### The Transition Table Editor window
You can edit your Transition Table in 2 locations: either (1) directly in the Inspector, by selecting the ScriptableObject asset in the Project window, or (2) with a custom-made Editor window which can be found under ChopChop > Transition Table Editor from the top menu. The window lists all Transition Tabels found in the project on the column to the left, for easy access.

This is what the Transition Table Editor window looks like:

![Transition Table Editor window](https://github.com/UnityTechnologies/open-project-1/blob/main/Docs/WikiImages/TransitionTable_PigChef.png)

And this is the same window while editing the Actions connected to a State:

![Transition Table Editor window showing actions](https://github.com/UnityTechnologies/open-project-1/blob/main/Docs/WikiImages/TransitionTable_PigChef_Actions.png)