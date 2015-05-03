**Version 1.2 - released 2013-03-30**
  * `Stop()` method that will exit the current state without entering a new one.
  * The state history is automatically trimmed to avoid that it grows indefinitely. The maximum number of entries in the state history can be defined using the `StateHistoryTrimThreshold` property.

---

**Version 1.1 - released 2013-03-26**
  * State history, which is an array of all the previous states that the state machine has been in
  * `GoBack` method to enable the state machine to go back to the latest state without the use of triggers.
  * New sample in the source solution: [WizardSample](https://code.google.com/p/solid-state/downloads/detail?name=WizardSample.1.1.0.zip), which demonstrates how a simple Windows Forms wizard can be built using each wizard page as a state.

---

**Version 1.0 - released 2013-03-21**
  * Initial version