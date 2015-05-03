The [TelephoneSample](https://code.google.com/p/solid-state/downloads/detail?name=TelephoneSample.1.2.0.zip) application is meant to demonstrate how a state machine can be set up with dedicated state classes and a [IoC](http://en.wikipedia.org/wiki/Inversion_of_control) container serving up states to enable dependency injection into them.

It is a .NET 4 Windows Forms application. The zip-file under Downloads only contains the executables, if you want to look at the source you must perform a [svn checkout](https://code.google.com/p/solid-state/source/checkout) and pull the entire solution.

# Points of interest #

The main setup action takes place in the `MainForm.cs` file, registering the necessary states and telephone parts in the Autofac container and configuring the state machine.

The `ValidTriggers` property is used to determine which trigger buttons should be enabled for each state.

The state classes uses different parts of the telephone (microphone, speaker, bell) to accomplish their tasks. The telephone parts emulate their behavior by writing to a textlog in the main form.

Nothing fancy, but I hopes the sample application serves its purpose of displaying how Solid.State can be used.