The [WizardSample](https://code.google.com/p/solid-state/downloads/detail?name=WizardSample.1.2.0.zip) application shows how Solid.State can use Windows Forms `UserControl` classes as states, letting each state be represented by a user control that gathers information in the wizard. The configuration of the state machine utilizes information from the wizard pages to determine the control flow through the state machine.

# Points of interest #
The main action of this sample application takes place in the `MainForm` class, configuring the machine and reacting to buttons that are clicked.

It also shows how the `GoBack()` method can be used to step back in the wizard without having to know the concrete target state. The state history of Solid.State is used for this purpose.

The pages of the wizard uses a common `WizardContext` instance to gather information, and this information is also used in some guard clauses in the state machine configuration to determine the flow of control.

This sample application also shows how dedicated and non-dedicated state classes can be mixed: The `ShutdownApplicationState` is a dedicated state class that exits the wizard after it has been finished or cancelled.