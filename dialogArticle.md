---
title: Event-driven uGUI Dialogs with node-based Editor
---

## Event-driven uGUI Dialogs with node-based Editor

This articles covers the creation of a simple general purpose dialog system, including a visual Editor that makes it easy to use for even complex dialog graphs. It is versatile: thanks to Unity's event widget, you can hook up any code to the dialog buttons. You can also form any shape of dialog connections: chains, trees, loops, graphs. The project lives on [GitHub][projectRepo].

### Data Setup

Let's define a dialog as a UI element that presents the player with some information and, optionally, a choice of replies. When the user selects a reply, there should be some effect on the game and the dialog UI should disappear. Translated to C# data types:

``` cs
public class DialogItem : MonoBehaviour
{
	public string text;
	public List<DialogReply> replies;
}

[Serializable]
public class DialogReply
{
	public string text;
	public DialogItem nextDialog;
	public Button.ButtonClickedEvent effect;
}
```

The dialog and each reply get a text field to present their information. As a visual improvement you could add fields for style, avatar image, positioning and such.
The reply class also has an effect field, which holds all gamestate changes that should happen when this reply is picked. 

We could store a "open that dialog" action as part of the effect event, but we want to be able to visualize the connections between dialogs and inspecting the internals of a UnityEvent is hard. That's why there's an extra field for it, called nextDialog.

Now you can put DialogItem components into empty GameObjects and fill them with content. This dialog turns another GameObject on/off:

<img src="https://raw.githubusercontent.com/seldomU/seldomu.github.io/master/pictures/DialogInspect.png" width="400">

### UI Setup

The Dialog UI is minimal: a panel with a vertical layout group, containing one Text for the question and enough Buttons to cover the maximum number of replies we expect. I chose 5.

<img src="https://raw.githubusercontent.com/seldomU/seldomu.github.io/master/pictures/DialogPanel.png" width="600">

### Hooking up data and UI

We need a manager component that can be used by the game to show and hide dialogs. Showing a dialog involves activating the panel and populating the question text and reply buttons from a given DialogItem. For hiding the dialog, we'll deactivate the panel. The full C# code is [here][dialogManagerSource]. Make sure not to put the manager on the panel's GameObject, or it will deactivate itself along with the panel. In the example project, the manager is attached to the Main Camera.

### Testing

Now we can feed DialogItems to the manager. The DialogTester component, attached to the Main Camera, opens dialogs with you press A or B and closes them when you press C.

* Dialog A lets you toggle the camera background image.
* Dialog B is a flowchart that helps the player to decide how to act in a hunger situation.

### Node-based editing

For things like conversations, you need to link many DialogItems together. The bigger such a graph gets, the harder it is to maintain and understand their structure. This is where node-based editors shine, and the [RelationsInspector][RIThread] lets us create one easily. We just need to supply a backend class, similar to Unity's custom inspectors. Then we can select it in the RelationsInspector and drag DialogItems into its window. The backend's main purpuse is to define what the graph nodes should represent, and how they are connected: 

``` cs
public class DialogBackend : MinimalBackend<DialogItem, string>
{
	public override IEnumerable<Relation<DialogItem, string>> GetRelations( DialogItem entity )
	{
		if ( entity.replies == null )
			yield break;

		foreach ( var reply in entity.replies )
		{
			if ( reply.nextDialog == null )
				continue;

			yield return new Relation<DialogItem, string>( entity, reply.nextDialog, reply.text );
		}
	}
}
```

The full implementation also adds context menu items for manipulating the dialog graph, you can see it here<link>. It is commented out because it dependds on the RelationsInspector [package][demopackage]. The example project's flowchart dialog graph looks like this:

<img src="https://raw.githubusercontent.com/seldomU/seldomu.github.io/master/pictures/DialogFlowChart.png" width="400">

That's it, now you're all set up for using dialogs in your projects.

[projectRepo]: https://github.com/seldomU/EventDrivenDialog
[demopackage]: https://www.assetstore.unity3d.com/en/#!/content/54884
[dialogManagerSource]: https://github.com/seldomU/EventDrivenDialog/blob/master/Assets/Scripts/DialogManager.cs
[RIThread]: http://forum.unity3d.com/threads/relationsinspector-reveal-structures-in-your-project-demo.382792/