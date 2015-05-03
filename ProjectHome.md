This is an implementation of a finite state machine that I need for my work. I also see it as a chance to make a first attempt at designing a fluent API.

One of the main features is that each state is implemented as a class implementing the `ISolidState` interface. This is an advantage over many other FSM implementations out there, and it separates the state logic from the state machine configuration in a nice way. Solid.State can be configured to either make the states singletons or create a new target state instance at each transition.

Solid.State also supports using an external state resolver whose only objective is to create state instances. Combine this with the IoC container of your choice and you can use dependency injection on your state classes.

Solid.State is heavily influenced by the [stateless](https://code.google.com/p/stateless/) project, which I've used extensively for the last few years to orchestrate asynchronous hardware communication.

**NOTE!** Solid.State sample applications are available for download on this page, but if you want to use it in your Visual Studio solution you should install using [Nuget](https://nuget.org/packages/Solid.State).

# Configuration #

Solid.State has an easy-to-use, fluent configuration API. Here's an example of how the famous telephone state machine can be set up:

```
var sm = new SolidMachine<TelephoneTrigger>();

sm.State<IdleState>()
    .IsInitialState()
    .On(TelephoneTrigger.PickingUpPhone).GoesTo<DiallingState>()
    .On(TelephoneTrigger.IncomingCall).GoesTo<RingingState>();

sm.State<RingingState>()
    .On(TelephoneTrigger.PickingUpPhone).GoesTo<ConversationState>()
    .On(TelephoneTrigger.IgnoreIncomingCall).GoesTo<IdleState>();

sm.State<DiallingState>()
    .On(TelephoneTrigger.FinishedDialling, () => !IsLineBusy).GoesTo<WaitForAnswerState>()
    .On(TelephoneTrigger.FinishedDialling, () => IsLineBusy).GoesTo<LineBusyState>()
    .On(TelephoneTrigger.MeHangingUp).GoesTo<IdleState>();

sm.State<LineBusyState>()
    .On(TelephoneTrigger.MeHangingUp).GoesTo<IdleState>();

sm.State<WaitForAnswerState>()
    .On(TelephoneTrigger.AnswerInOtherEnd).GoesTo<ConversationState>()
    .On(TelephoneTrigger.MeHangingUp).GoesTo<IdleState>();

sm.State<ConversationState>()
    .On(TelephoneTrigger.MeHangingUp).GoesTo<IdleState>()
    .On(TelephoneTrigger.OtherEndHangingUp).GoesTo<LineDisconnectedState>();

sm.State<LineDisconnectedState>()
    .On(TelephoneTrigger.MeHangingUp).GoesTo<IdleState>();

sm.Start();
```

Starting the state machine will trigger a `Transitioned` event where the `SourceState` property is null.

## Setting the initial state ##

By default, the first configured state becomes the initial state, e.g. `IdleState` in the code above. You can define another initial state by using the `IsInitialState()` method:

```
sm.State<RingingState>()
    .IsInitialState()
    .On(TelephoneTrigger.PickingUpPhone).GoesTo<ConversationState>()
    .On(TelephoneTrigger.NotAnswering).GoesTo<IdleState>();
```

Only one state can be defined as the initial state, otherwise an exception will be thrown.


## Defining a context ##

The state machine can be configured to use a context that is fed to the states in the `Entering` and `Exiting` methods. This is a convenient way to make relevant data available to the state classes without having to inject it.

The context can be defined either in the constructor or by setting the `Context` property:

```
var sm = new SolidMachine<TelephoneTrigger>(_applicationController);

var sm2 = new SolidMachine<TelephoneTrigger>();
sm2.Context = _applicationController;
```

If no context is explicitly set, the state machine will use itself as the context.


## Reentrant states ##

The state machine can be configured to allow reentrant states, which means a state that transitions to/from itself. During the transition, the `Exiting` and `Entering` methods of the state will be triggered.

The configuration is simply done by defining a transition that points back to the same state:

```
sm.State<SendEmailState>()
    .On(Trigger.ResendEmailRequested).GoesTo<SendEmailState>();
```


# Guard clauses #

You can control the flow of the state machine by configuring guard clauses in the configuration. A guard clause is basically a `Func<bool>` that returns either True or False.

```
var sm = new SolidMachine<TelephoneTrigger>();
sm.State<IdleState>()
    .On(TelephoneTrigger.PickingUpPhone, () => IsPhoneWorking).GoesTo<DiallingState>()
    .On(TelephoneTrigger.PickingUpPhone, () => !IsPhoneWorking).GoesTo<TelephoneBrokenState>()
    .On(TelephoneTrigger.IncomingCall).GoesTo<RingingState>();
```

Be sure to always only have one guard clause for a state/trigger combination that evaluates to True at any given point, otherwise a runtime exception will be thrown.


# Triggering a transition #

To fire a trigger on the state machine, simply call the `Trigger` method:

```
sm.Trigger(TelephoneTrigger.PickingUpPhone);
```

The `ValidTriggers` property can be used to query the state machine for a list of the triggers that are valid for the current state:

```
foreach (var trigger in sm.ValidTriggers)
    Console.WriteLine("Valid trigger: " + trigger.ToString());
```


# Stopping the state machine #

The state machine also has a `Stop()` method. Calling this method will make the current state exit without entering a new state. This is good practice since the current state may have code in the `Exiting` method that releases resources and does other cleanup.

The call to the `Stop()` method will trigger a `Transitioned` event where `TargetState = null`.


# Creating states #

A state for the state machine can be any class implementing the `ISolidState` interface.

This means that you can choose between writing dedicated state classes or make existing classes implement the ISolidState interface. For instance, it would be possible to build a wizard where each page is a `WinForms` `UserControl` that also implements the `ISolidState ` interface.

A simple dedicated state class:

```
public class AnnoyingSoundState : ISolidState
{
    public void Entering(object context)
    {
        StartAnnoyingSound();
    }

    public void Exiting(object context)
    {
        StopAnnoyingSound();
    }
}
```



# Resolving states #

When a transition is under way, the state machine needs to make sure that the target state object gets instantiated. The default behavior for Solid.State is to use the standard .NET activation mechanism to instantiate the correct state.

Solid.State can be configured to either treat the states as singletons, or to create new target state instances on each transition.

The behavior is configured through the `StateInstantiationMode` property. It is by default set to `StateInstantiationMode.Singleton`. To get a new state instance on each transition, configure the state machine before it is started:

```
var sm = new SolidMachine<TelephoneTrigger>();
sm.StateInstantiationMode = StateInstantiationMode.PerTransition;
```

## Using a state resolver ##

Solid.State can be configured to use a state resolver, which is then responsible for instantiating state objects depending on the type. This plays very well with an IoC container, which makes it possible to use constructor or property injection on the state classes.

The state resolver must implement the `IStateResolver` interface, which is very simple:
```
public interface IStateResolver
{
    ISolidState ResolveState(Type stateType);
}
```

The state resolver can either be set in the constructor or by assigning it to the `StateResolver` property:

```
var stateMachine = new SolidMachine<TelephoneTrigger>(null, this);

var stateMachine2 = new SolidMachine<TelephoneTrigger>();
stateMachine2.StateResolver = _autofacResolver;
```

My preference for IoC is Autofac, which could make the resolver look something like this:

```
public class MyStateResolver : IStateResolver
{
    private readonly IComponentContext _context;

    // Constructor

    public MyStateResolver(IComponentContext context)
    {
        _context = context;
    }

    // Methods (IStateResolver)

    public ISolidState ResolveState(Type stateType)
    {
        return (ISolidState) _context.Resolve(stateType);
    }
}
```


## State resolver vs. state instantiation mode ##

The state resolver and the state machine can sabotage for eachother in certain scenarios:
  * If the state machine is set to `StateInstantiationMode = PerTransition` but the state resolver delivers singletons, then the state objects will be singletons.
  * If the state machine is set to `StateInstantiationMode = Singleton` but the state resolver is configured to return new instances on each request, the state objects will still be singletons. This is because the state machine only will ask for state instances once and then keep references to them.


# Listening in #

To keep track of what's happening inside the state machine, the `Transitioned` event can be used. This is called when a transition has occured, defining the source and target state of the transition:

```
var sm = new SolidMachine<TelephoneTrigger>();
sm.Transitioned += SolidMachineOnTransitioned;

...

private void SolidMachineOnTransitioned(object sender, TransitionedEventArgs eventArgs)
{
    tbLog.Text += string.Format("Transitioned from '{0}' to '{1}'...", eventArgs.SourceState, eventArgs.TargetState);
}
```

Be aware that either the `SourceState` or `TargetState` can be null when the state machine is starting or stopping.

NOTE! When this event is raised, the `Exiting` method on the source state and the `Entering` method on the target state has been executed already.

## The current state ##

The `CurrentState` property can be used to query the current state of the state machine.

```
if (sm.CurrentState is ConversationState)
    MessageBox.Show("Stop talking, I need the phone!");
```