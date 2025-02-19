
---
layout: page
title: Developer Guide
---
* Table of Contents
  {:toc}

--------------------------------------------------------------------------------------------------------------------

## **Acknowledgements**

* {list here sources of all reused/adapted ideas, code, documentation, and third-party libraries -- include links to the original source as well}

--------------------------------------------------------------------------------------------------------------------

## **Setting up, getting started**

Refer to the guide [_Setting up and getting started_](SettingUp.md).

--------------------------------------------------------------------------------------------------------------------

## **Design**

<div markdown="span" class="alert alert-primary">

:bulb: **Tip:** The `.puml` files used to create diagrams in this document `docs/diagrams` folder. Refer to the [_PlantUML Tutorial_ at se-edu/guides](https://se-education.org/guides/tutorials/plantUml.html) to learn how to create and edit diagrams.
</div>

### Architecture

<img src="images/ArchitectureDiagram.png" width="280" />

The ***Architecture Diagram*** given above explains the high-level design of the App.

Given below is a quick overview of main components and how they interact with each other.

**Main components of the architecture**

**`Main`** (consisting of classes [`Main`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/Main.java) and [`MainApp`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/MainApp.java)) is in charge of the app launch and shut down.
* At app launch, it initializes the other components in the correct sequence, and connects them up with each other.
* At shut down, it shuts down the other components and invokes cleanup methods where necessary.

The bulk of the app's work is done by the following four components:

* [**`UI`**](#ui-component): The UI of the App.
* [**`Logic`**](#logic-component): The command executor.
* [**`Model`**](#model-component): Holds the data of the App in memory.
* [**`Storage`**](#storage-component): Reads data from, and writes data to, the hard disk.

[**`Commons`**](#common-classes) represents a collection of classes used by multiple other components.

**How the architecture components interact with each other**

The *Sequence Diagram* below shows how the components interact with each other for the scenario where the user issues the command `delete 1`.

<img src="images/ArchitectureSequenceDiagram.png" width="574" />

Each of the four main components (also shown in the diagram above),

* defines its *API* in an `interface` with the same name as the Component.
* implements its functionality using a concrete `{Component Name}Manager` class (which follows the corresponding API `interface` mentioned in the previous point.

For example, the `Logic` component defines its API in the `Logic.java` interface and implements its functionality using the `LogicManager.java` class which follows the `Logic` interface. Other components interact with a given component through its interface rather than the concrete class (reason: to prevent outside component's being coupled to the implementation of a component), as illustrated in the (partial) class diagram below.

<img src="images/ComponentManagers.png" width="300" />

The sections below give more details of each component.

### UI component

The **API** of this component is specified in [`Ui.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/ui/Ui.java)

![Structure of the UI Component](images/UiClassDiagram.png)

The UI consists of a `MainWindow` that is made up of parts e.g.`CommandBox`, `ResultDisplay`, `PersonListPanel`, `StatusBarFooter` etc. All these, including the `MainWindow`, inherit from the abstract `UiPart` class which captures the commonalities between classes that represent parts of the visible GUI.

The `UI` component uses the JavaFx UI framework. The layout of these UI parts are defined in matching `.fxml` files that are in the `src/main/resources/view` folder. For example, the layout of the [`MainWindow`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/ui/MainWindow.java) is specified in [`MainWindow.fxml`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/resources/view/MainWindow.fxml)

The `UI` component,

* executes user commands using the `Logic` component.
* listens for changes to `Model` data so that the UI can be updated with the modified data.
* keeps a reference to the `Logic` component, because the `UI` relies on the `Logic` to execute commands.
* depends on some classes in the `Model` component, as it displays `Person` object residing in the `Model`.

### Logic component

**API** : [`Logic.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/logic/Logic.java)

Here's a (partial) class diagram of the `Logic` component:

<img src="images/LogicClassDiagram.png" width="550"/>

The sequence diagram below illustrates the interactions within the `Logic` component, taking `execute("delete 1")` API call as an example.

![Interactions Inside the Logic Component for the `delete 1` Command](images/DeleteSequenceDiagram.png)

<div markdown="span" class="alert alert-info">:information_source: **Note:** The lifeline for `DeleteCommandParser` should end at the destroy marker (X) but due to a limitation of PlantUML, the lifeline reaches the end of diagram.
</div>

How the `Logic` component works:

1. When `Logic` is called upon to execute a command, it is passed to an `AddressBookParser` object which in turn creates a parser that matches the command (e.g., `DeleteCommandParser`) and uses it to parse the command.
1. This results in a `Command` object (more precisely, an object of one of its subclasses e.g., `DeleteCommand`) which is executed by the `LogicManager`.
1. The command can communicate with the `Model` when it is executed (e.g. to delete a person).
1. The result of the command execution is encapsulated as a `CommandResult` object which is returned back from `Logic`.

Here are the other classes in `Logic` (omitted from the class diagram above) that are used for parsing a user command:

<img src="images/ParserClasses.png" width="600"/>

How the parsing works:
* When called upon to parse a user command, the `AddressBookParser` class creates an `XYZCommandParser` (`XYZ` is a placeholder for the specific command name e.g., `AddCommandParser`) which uses the other classes shown above to parse the user command and create a `XYZCommand` object (e.g., `AddCommand`) which the `AddressBookParser` returns back as a `Command` object.
* All `XYZCommandParser` classes (e.g., `AddCommandParser`, `DeleteCommandParser`, ...) inherit from the `Parser` interface so that they can be treated similarly where possible e.g, during testing.

### Model component
**API** : [`Model.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/model/Model.java)

<img src="images/ModelClassDiagram.png" width="450" />


The `Model` component,

* stores the address book data i.e., all `Person` objects (which are contained in a `UniquePersonList` object).
* stores the currently 'selected' `Person` objects (e.g., results of a search query) as a separate _filtered_ list which is exposed to outsiders as an unmodifiable `ObservableList<Person>` that can be 'observed' e.g. the UI can be bound to this list so that the UI automatically updates when the data in the list change.
* stores a `UserPref` object that represents the user’s preferences. This is exposed to the outside as a `ReadOnlyUserPref` objects.
* does not depend on any of the other three components (as the `Model` represents data entities of the domain, they should make sense on their own without depending on other components)

<div markdown="span" class="alert alert-info">:information_source: **Note:** An alternative (arguably, a more OOP) model is given below. It has a `Tag` list in the `AddressBook`, which `Person` references. This allows `AddressBook` to only require one `Tag` object per unique tag, instead of each `Person` needing their own `Tag` objects.<br>

<img src="images/BetterModelClassDiagram.png" width="450" />

</div>


### Storage component

**API** : [`Storage.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/storage/Storage.java)

<img src="images/StorageClassDiagram.png" width="550" />

The `Storage` component,
* can save both address book data and user preference data in JSON format, and read them back into corresponding objects.
* inherits from both `AddressBookStorage` and `UserPrefStorage`, which means it can be treated as either one (if only the functionality of only one is needed).
* depends on some classes in the `Model` component (because the `Storage` component's job is to save/retrieve objects that belong to the `Model`)

### Common classes

Classes used by multiple components are in the `seedu.addressbook.commons` package.

--------------------------------------------------------------------------------------------------------------------

## **Implementation**

This section describes some noteworthy details on how certain features are implemented.

### Undo/redo feature

The undo/redo mechanism is facilitated by `TrackedAddressBook`. It extends `AddressBook` with an undo/redo history, stored internally as an `addressBookStateList` and `currentStatePointer`. 
Each `AddressBook` also stores a copy of `ShortcutSettings` that matches the shortcuts that the user is using at that frame.
Additionally, it implements the following operations:

* `TrackedAddressBook#commit()` — Saves the current address book state in its history.
* `TrackedAddressBook#undo()` — Restores the previous address book state from its history.
* `TrackedAddressBook#redo()` — Restores a previously undone address book state from its history.

These operations are exposed in the `Model` interface as `Model#commitAddressBook()`, `Model#undoAddressBook()` and `Model#redoAddressBook()` respectively.

Given below is an example usage scenario and how the undo/redo mechanism behaves at each step.

Step 1. The user launches the application for the first time. The `TrackedAddressBook` will be initialized with the initial address book state, and the `currentStatePointer` pointing to that single address book state.

![UndoRedoState0](images/UndoRedoState0.png)

Step 2. The user executes `delete 5` command to delete the 5th person in the address book. The `delete` command calls `Model#commitAddressBook()`, causing the modified state of the address book after the `delete 5` command executes to be saved in the `addressBookStateList`, and the `currentStatePointer` is shifted to the newly inserted address book state.

![UndoRedoState1](images/UndoRedoState1.png)

Step 3. The user executes `add n/David …​` to add a new person. The `add` command also calls `Model#commitAddressBook()`, causing another modified address book state to be saved into the `addressBookStateList`.

![UndoRedoState2](images/UndoRedoState2.png)

<div markdown="span" class="alert alert-info">:information_source: **Note:** If a command fails its execution, it will not call `Model#commitAddressBook()`, so the address book state will not be saved into the `addressBookStateList`.

</div>

Step 4. The user now decides that adding the person was a mistake, and decides to undo that action by executing the `undo` command. The `undo` command will call `Model#undoAddressBook()`, which will shift the `currentStatePointer` once to the left, pointing it to the previous address book state, and restores the address book to that state.

![UndoRedoState3](images/UndoRedoState3.png)

<div markdown="span" class="alert alert-info">:information_source: **Note:** If the `currentStatePointer` is at index 0, pointing to the initial AddressBook state, then there are no previous AddressBook states to restore. The `undo` command uses `Model#canUndoAddressBook()` to check if this is the case. If so, it will return an error to the user rather
than attempting to perform the undo.

</div>

The following sequence diagram shows how the undo operation works:

![UndoSequenceDiagram](images/UndoSequenceDiagram.png)

<div markdown="span" class="alert alert-info">:information_source: **note:** the lifeline for `undocommand` should end at the destroy marker (x) but due to a limitation of plantuml, the lifeline reaches the end of diagram.

</div>

The `redo` command does the opposite — it calls `Model#redoAddressBook()`, which shifts the `currentStatePointer` once to the right, pointing to the previously undone state, and restores the address book to that state.

<div markdown="span" class="alert alert-info">:information_source: **Note:** If the `currentStatePointer` is at index `addressBookStateList.size() - 1`, pointing to the latest address book state, then there are no undone AddressBook states to restore. The `redo` command uses `Model#canRedoAddressBook()` to check if this is the case. If so, it will return an error to the user rather than attempting to perform the redo.

</div>

Step 5. The user then decides to execute the command `list`. Commands that do not modify the address book, such as `list`, will usually not call `Model#commitAddressBook()`, `Model#undoAddressBook()` or `Model#redoAddressBook()`. Thus, the `addressBookStateList` remains unchanged.

![UndoRedoState4](images/UndoRedoState4.png)

Step 6. The user executes `clear`, which calls `Model#commitAddressBook()`. Since the `currentStatePointer` is not pointing at the end of the `addressBookStateList`, all address book states after the `currentStatePointer` will be purged. Reason: It no longer makes sense to redo the `add n/David …​` command. This is the behavior that most modern desktop applications follow.

![UndoRedoState5](images/UndoRedoState5.png)

The following activity diagram summarizes what happens when a user executes a new command:

<img src="images/CommitActivityDiagram.png" width="250" />

<br>

### View Command Feature

The view command feature allows the user to view the details of a specific patient or specialist.

Given below is an example usage scenario and how the view mechanism behaves at each step.

Step 1. The user launches the application. The List of person will be initialized with the initial
state, and the `Current Selected Person` pointer pointing to the first person on the list. <br>

![ViewState1](images/ViewState1.png)


Step 2. The user executes `view 2` command to view the 2nd person details in the `DoConnek Pro`.
The `view` command update the `Current Selected Person` pointer with the corresponding index from the
input. <br>

![ViewState2](images/ViewState2.png)


Step 3. The UI now updates the view panel to display the newly selected person and their details.

The sequence diagram below shows how the view operation works:
![ViewSequenceDiagram](images/ViewSequenceDiagram.png)

The following activity diagram summarizes what happens when a user executes the view command:
![ViewActivityDiagram](images/ViewActivityDiagram.png)

<br>

### Recall recent commands feature

The recent command feature is facilitated by the `CommandStringStash`. This is a stash that stores the history
of the command string of the 20 most recent commands executed. Internally, it is stored as a `cmdStringStack`,
and `currentCmdIndex`. This internal representation allows cycling through the recent commands both forwards
and backwards.

These operations are exposed in the `Logic` interface as `Logic#getPrevCommandString(String commandInputString)`,
`Logic#getPassedCommandString(String commandInputString)`, and `Logic#addCommandString(String commandInputString)`.

The following operations are implemented by the `CommandStringStash`:
* `CommandStringStash#addCommandString(String commandInputString)` - Adds `commandInputString` to the history.
* `CommandStringStash#getPrevCommandString(String commandInputString)` - Cycles one command further back in history.
* `CommandStringStash#getPassedCommandString(String commandInputString)` - Cycles one command further forward in history.

<div markdown="span" class="alert alert-info">:information_source: **note:** Cycling fowards or backwards may not always be
valid operations. No cycling forward or backward can be done if the stash is empty. No cycling backward
can be done if the user is already on the least recent command in the stash, and no cycling forward can be done
if the user has not yet cycled backward. To consider all these cases, the `commandInputString` is passed as a parameter
to `CommandStringStash#getPrevCommandString(String commandInputString)` and `CommandStringStash#getPassedCommandString(String commandInputString)`.
The `commandInputString` is the current command in the CLI textbox and is returned from these methods in the case of invalid operations
so there is no change to the CLI textbox.

</div>

Given below is an example usage scenario and how the recall recent commands feature works at each step.

Step 1. The user launches the application for the first time. The `CommandStringStash` will be initialised
with no command strings and a `currentCmdIndex` of 0.

![Recall Step 1](images/RecallStep1.png)

Step 2. The user executes the `list -sp` command to list the specialists in DoConnek Pro. Upon success,
`Logic#addCommandString("list -sp")` is called, adding this command string to the `CommandStringStash` and moving the
`currentCmdIndex` to point to right after the element added.

![Recall Step 2](images/RecallStep2.png)

<div markdown="span" class="alert alert-info">:information_source: **note:** After an addition, the `currentCmdIndex` is set to point
one index after the last element in the `CommandStringStash`. The `Logic#getPrevCommandString` method decrements
this index before returning the String pointed to by `currentCmdIndex` so this works as a way of 'resetting' the state
allowing the user to start to cycle back from the most recently added command again.

</div>

Step 3. The user executes two more commands `help` and `delete 1` in the respective order. As before, the
`CommandStringStash` is updated appropriately.

![Recall Step 3](images/RecallStep3.png)

The following activity diagram summarises how the `CommandStringStash` is updated when a user executes a command.

![Add Command String](images/AddCommandStringActivityDiagram.png)

Step 4. The user wants to list the patients in DoConnek Pro but forgot how to do so. They decide to execute the `help`
command. To do so efficiently, they press the up arrow on the keyboard to recall the `help` command they recently entered.
This results in `Logic#getPrevCommandString` being called which returns `delete 1`.  The user's CLI text box is then set to display `delete 1`.

![Recall Step 4](images/RecallStep4.png)

The following sequence diagram shows how this recalling of the previous command string works.

![Recall Command Sequence Diagram](images/RecallCommandSequenceDiagram.png)

To cycle forward in history, a similar sequence is followed, but `Logic#getPassedCommandString` and its
corresponding methods are called instead.

Step 5. The user presses the up arrow again, and this time `Logic#getPrevCommandString` returns `help` which is displayed
on the user's CL. They can now easily execute `help`.

![Recall Step 5](images/RecallStep5.png)

The following activity diagram summarises what happens when a user presses the up arrow.

![Up Arrow Activity Diagram](images/UpArrowActivityDiagram.png)

Step 6. Instead of entering `help` the user decides to delete the first specialist currently displayed.
They press the down arrow on the keyboard to recall the `delete 1` command they just passed.
This results in `Logic#getPassedCommandString` being called which returns `delete 1`.  The user's CLI text box is then set to display `delete 1`.

![Recall Step 6](images/RecallStep6.png)

<br>

### Find feature

#### Overview
The find feature allows the user to find patients or specialists by checking whether their attributes contain certain keywords.
Upon entering a `find` command, an instance of `FindCommandParser` is created to process the prefixes along with their corresponding arguments into
predicates. The predicates are represented as `Predicate<Person>` (using Java's in-built functional interface), and are mapped
to their prefixes in a `FindPredicateMap`. A `FindPredicateMap` encapsulates **all** predicates indicated by the user,
which are later combined and used to test each `Person` in the `FilteredPersonList` of the `Model`.

As patients and specialists have common and differing attributes, so do their predicates.<br>
Predicates common to both:<br>
`NameContainsKeywordsPredicate`, `PhoneContainsKeywordsPredicate`, `EmailContainsKeywordsPredicate`, `TagsContainsKeywordsPredicate`

Predicates unique to patients:<br>
`AgeContainsKeywordsPredicate`, `MedHistoryContainsKeywordsPredicate`

Predicates unique to specialist:<br>
`LocationContainsKeywordsPredicate`, `SpecialtyContainsKeywordsPredicate`

#### Example
The following sequence diagram shows how a find command is parsed and executed to find a patient.
In this example, the command entered is <br>
`find -pa n/Tim m/Anaemia`

![FindCommandSequenceDiagram](images/FindCommandSequenceDiagram.png)

When the `FindCommandParser` parses the prefix arguments, a `NameContainsKeywordsPredicate` and `MedHistoryContainsKeywordsPredicate`
is instantiated by their corresponding arguments and are mapped to their prefixes in the `FindPredicateMap`.

![FindPredicateMapExample](images/FindPredicateMapExample.png)

The predicates are combined into a single `Predicate<Person>` in `FindCommand#execute` and applied to each `Person` in the 
`FilteredPersonList` of the `Model`.

To find a specialist, a similar parse and execution flow is conducted. 

<br>

### Shortcut management feature

User defined shortcuts are managed by `ShortcutSettings`. Internally it contains a `shortcutMap` that stores _mappings_ of 
user defined _shortcut aliases_ to existing valid _command keywords_. This class provides functionality for registering new shortcuts,
removing previously defined shortcuts, and querying the map to see check if a shortcut has previously been defined.

These shortcut mappings need to be updated by command execution, as well as used in parsing of user input. Thus, the following design decisions have been made:
1. Shortcut operations are exposed in the `Model` interface as 
   * `Model#registerShortcut(ShortcutAlias shortcutAlias, CommandWord commandWord)` 
   * `Model#removeShortcut(ShortcutAlias shortcutAlias)`
   * `Model#getShortcut(String alias)`
2. `AddressBookParser` now has a dependency to the `Model`. This is required so that the shortcut mappings can be accessed for use in parsing user input.

![Class Diagram](images/ShortcutSettingsClassDiagram.png)

Within `ShortcutSettings` similar operations are implemented to `register`, `remove`, and `get` shortcuts. Additionally to resolve bugs that might arise from users tampering with the `preferences.json` file,
`ShortcutSettings#removeBadMappings()` clears the `shortcutMap` of invalid mappings and is called on initialisation.

<div markdown="block" class="alert alert-info">

#### Invalid Mappings
On this subject, in order to not jeopardise parsing operations and the user experience, some restrictions have been put on the mappings that can be registered. 
These restrictions are enforced by the wrapper class `ShortcutAlias`, along with the relevant method in `ParserUtil`.
1. Shortcut aliases must not be blank.
2. Shortcut aliases must only contain alphanumeric characters without any whitespaces.
3. Shortcut aliases must not match the default `COMMAND_WORD` of any existing commands.
   * This is to prevent any unintended behaviour, and users might even accidentally 'lock' themselves out of using certain commands if they are unaware of how to manipulate `preferences.json`.
</div>

#### Command execution
The sequence diagram below outlines the command execution of a sample `AddShortcutCommand` that wants to register `del` as a shortcut for `delete`.

![Add Shortcut Sequence](images/AddShortcutCommandSequenceDiagram.png)

The following activity diagrams summarise the process of adding and removing shortcuts from DoConnek Pro.

![Add Shortcut Activity](images/AddShortcutActivityDiagram.png)
![Delete Shortcut Activity](images/DeleteShortcutActivityDiagram.png)

#### Saving between sessions
ShortcutSettings implements the `Serializable` interface, thus is saved to `json` format as a part of `UserPrefs`. 

#### Design considerations:
**Aspect: How shortcuts are stored and accessed:**

* **Alternative 1 (current choice):** Stored in a separate `ShortcutSettings` class.
    * Pros: Separation of responsibility and easier management.
    * Cons: More memory usage and introduces one dependency between `AddressBookParser` and `Model`.

* **Alternative 2:** Individual command knows its own list of shortcuts.
    * Pros: Will use less memory (No extra data structure created).
    * Cons: Difficult to manage duplicate shortcut mappings.

--------------------------------------------------------------------------------------------------------------------

## **Planned Enhancements**

1. DoConnek Pro currently checks for duplicate persons by name. This means that people with the same names cannot be added even if they have different parameters (like `Phone` or `Email`). 
We plan on implementing an `NRIC` field for patients and an `MCR` field for specialists as unique identifiers to solve this issue. 

--------------------------------------------------------------------------------------------------------------------

## **Documentation, logging, testing, configuration, dev-ops**

* [Documentation guide](Documentation.md)
* [Testing guide](Testing.md)
* [Logging guide](Logging.md)
* [Configuration guide](Configuration.md)
* [DevOps guide](DevOps.md)

--------------------------------------------------------------------------------------------------------------------

## **Appendix: Requirements**

### Product scope

**Target user profile**:
The target users...

* are management staff at a General Practitioner's Clinic
* need to keep track of the clinic's patients' details
* need to keep track of the affiliated specialists that the clinic refers patients to
* prefer desktop apps over other types
* can type fast
* prefer typing to mouse interactions
* are reasonably comfortable using CLI apps

**Value proposition**: A one-stop application that allows management of both a clinic's patient and specialist details faster than a typical mouse/GUI driven app.



### User stories

Priorities: Essential (must have) - `* * *`, Typical (nice to have) - `* *`, Novel (unlikely to have) - `*`

| Priority | As a …​                  | I want to …​                                        | So that I can…​                                                      |
|---------|--------------------------|-----------------------------------------------------|----------------------------------------------------------------------|
| `* * *` | new user                 | see usage instructions                              | refer to instructions when I forget how to use the App               |
| `* * *` | user                     | deal with patient and specialist records separately | easily do patient-specific or specialist-specific operations         |
| `* * *` | user                     | add a new patient to the records                    | keep track of details of new patients                                |
| `* *`   | user                     | edit details of existing patients                   | make sure the patients' details are up to date                       |
| `* * *` | user                     | delete individual patient details                   | delete records of patients I no longer take care                     |
| `* *`   | user                     | easily filter patient records by any criteria       | find relevant patient information quickly                            |
| `* * *` | user                     | access a patient's name                          | identify the patient                                              |
| `* * *` | user                     | access a patient's medical history                  | provide more informed care based on the patient's medical history    |
| `* * *` | user                     | access a patient's contact number                   | quickly get in contact with the patient                              |
| `* * *` | user                     | access a patient's email                            | have an alternate means of contacting the patient                    |
| `* * *` | user                     | access a patient's age                              | provide age-specific care and treatment to the patient               |
| `*`     | user                     | customise tags for my patients                      | have control over the organisation of my patients                    |
| `*`     | user                     | group delete patients                               | save time when removing large amount of patient records              |
| `* * *` | user                     | add a new specialist to the records                 | keep track of details of new specialists patients can be referred to |
| `* *`   | user                     | edit details of existing specialists                | make sure the specialist details are up to date                      |
| `* * *` | user                     | delete specialist details                           | delete records of specialists I no longer work with                  |
| `* *`   | user                     | easily filter specialist records by any criteria       | find relevant specialist information quickly                            |
| `* * *` | user                     | access a specialist's name                          | identify the specialist                                              |
| `* * *` | user                     | access a specialist's contact number                | quickly get in contact with the specialist                           |
| `* * *` | user                     | access a specialist's email                         | have an alternate means of contacting the specialist                 |
| `* * *` | user                     | access a specialist's specialty                     | refer patients to appropriate specialists                            |
| `*`     | user                     | customise tags for my specialists                   | have control over the organisation of my specialists                 |
| `* *`   | user                     | search for specialists by location                  | refer patients to convenient specialists to minimise time            | 
| `* *`   | user                     | search for specialists by their speciality          | refer patients to the right experts                                  |
| `*`     | user                     | customise colour schemes of the App                 | customise the look of my application                                 |
| `*`     | user who is a fast typer | perform all tasks will the CLI rather than the GUI  | be more efficient in managing records                                |
| `* *`   | impatient user           | add new keyboard shortcuts to the App               | save time when performing frequently repeated tasks                  |
| `* *`   | impatient user           | recall recently executed commands                   | save time when performing frequently repeated tasks                  |
| `* *`   | careless user            | undo and redo commands                              | correct mistakes I've made                                           |
| `* * *` | user                     | save the App data                                   | retain data for future reference                                     |
| `* * *` | user                     | load previously saved App data                      | examine past records/information                                     |
| `*`     | experienced user         | manipulate the save file directly                   | make quick changes to data without having to open the application    |

### Use cases

(For all use cases below, the **System** is the `DoConnek Pro` and the **Actor** is the `user`, unless specified otherwise)

**Use case: Delete a person**

**MSS**

1.  User requests to list persons
2.  System shows a list of persons
3.  User requests to delete a specific person in the list
4.  System deletes the person

    Use case ends.

**Extensions**

* 2a. The list is empty.

  Use case ends.

* 3a. The given index is invalid.

    * 3a1. System shows an error message.

      Use case resumes at step 2.

**Use case: Add a patient**

**MSS**

1. User searches list to check if patient is already in the system
2. System shows that there are no entries matching the patient's name
3. User adds patient to the system
4. System confirms that the person has been added


    Use case ends.

**Extensions**

* 3a. The given parameters are invalid.

  * 3a1. System shows an error message.

    Use case resumes at step 2.

**Use case: Listing all patients**

**MSS**

1. User requests to list all patients
2. System shows a list of all patients stored 
   
    Use case ends.

**Extensions**

* 1a. System detects invalid request format

    * 1a1. System shows an error message.
  
      Use case ends.

**Use case: Listing all specialists**

**MSS**

1. User requests to list all specialists
2. System shows a list of all specialists stored 

    Use case ends.

**Extensions**

* 1a. System detects invalid request format

    * 1a1. System shows an error message.

      Use case ends.

**Use case: Searching for patients**

**MSS**

1. User requests to find patients from a keyword.
2. System shows a list of patients that match the keyword with their names.

   Use case ends.

**Extensions**

* 1a. System detects invalid request format

    * 1a1. System shows an error message.

      Use case ends.

* 1b. System could not find any patient from the keyword

    * 1b1. System shows an error message.

      Use case ends.

**Use case: View a person**

**MSS**

1.  User requests to list persons
2.  System shows a list of persons
3.  User requests to view the details a specific person in the list
4.  System shows the person details

    Use case ends.

**Extensions**

* 2a. The list is empty.

  Use case ends.

* 3a. The given index is invalid.

    * 3a1. System shows an error message.

      Use case resumes at step 2.

**Use case: Exit the program**

**MSS**

1.  User requests to exit the program
2.  System exits the program

* 1a. System detects invalid request format.

  * 1a1. System shows an error message.

    Use case ends.

*{More to be added}*

### Non-Functional Requirements

1.  Should work on any _mainstream OS_ as long as it has Java `11` or above installed.
2.  Should be able to hold up to 1000 persons without a noticeable sluggishness in performance for typical usage.
3.  A user with above average typing speed for regular English text (i.e. not code, not system admin commands) should be able to accomplish most of the tasks faster using commands than using the mouse.
4.  Should be made for use by a single user rather than multiple users.
5.  Any data should be stored locally in a human-editable text file without any use of a Database Management System.
6.  Should be usable with the download of a single JAR file of size less than 100 MB.
7.  Should not require any installation by the users.
7.  Should not depend on any remote server.
8.  The product should start from the given code base and be evolved/enhanced/morphed in a breadth-first incremental manner.
9.  The code should primarily follow the Object-oriented paradigm.
10. Should only use _appropriate third party frameworks/libraries/services_ that have been approved.
11. The product's _GUI should work well_ for standard screen resolutions 1920x1080 and higher, and, for screen scales 100% and 125%.
12. The product's _GUI should be usable_ for resolutions 1280x720 and higher, and, for screen scales 150%.
13. Should be intuitive to use for a user without a technical background.
13. The product is not required to handle printing of the patient and specialist records.


### Glossary

* **Mainstream OS**: Windows, Linux, Unix, OS-X
* **Private contact detail**: A contact detail that is not meant to be shared with others
* **Appropriate third party frameworks/libraries/systems**: Those frameworks/libraries/systems that are free, open-source, have permissive license terms, don't require installation by the users and don't violate other constraints.
* **GUI should work well**: The GUI should not cause resolution-related inconveniences for the user.
* **GUI should be usable**: The GUI's functions can all be used, although user-experience may be suboptimal.

--------------------------------------------------------------------------------------------------------------------

## **Appendix: Instructions for manual testing**

Given below are instructions to test the app manually.

<div markdown="span" class="alert alert-info">:information_source: **Note:** These instructions only provide a starting point for testers to work on;
testers are expected to do more *exploratory* testing.

</div>

### Launch and shutdown

1. Initial launch

    1. Download the jar file and copy into an empty folder

    1. Double-click the jar file Expected: Shows the GUI with a set of sample contacts. The window size may not be optimum.

1. Saving window preferences

    1. Resize the window to an optimum size. Move the window to a different location. Close the window.

    1. Re-launch the app by double-clicking the jar file.<br>
       Expected: The most recent window size and location is retained.

1. _{ more test cases …​ }_

### Deleting a person

1. Deleting a person while all persons are being shown

    1. Prerequisites: List all persons using the `list` command. Multiple persons in the list.

    1. Test case: `delete 1`<br>
       Expected: First contact is deleted from the list. Details of the deleted contact shown in the status message. Timestamp in the status bar is updated.

    1. Test case: `delete 0`<br>
       Expected: No person is deleted. Error details shown in the status message. Status bar remains the same.

    1. Other incorrect delete commands to try: `delete`, `delete x`, `...` (where x is larger than the list size)<br>
       Expected: Similar to previous.

1. _{ more test cases …​ }_

### Viewing a person

1. Viewing a person while all persons are being shown

    1. Prerequisites: List all persons using the `list` command. Multiple persons in the list.

    1. Test case: `view 1`<br>
       Expected: First person is selected to be viewed. Details of the viewed person shown in the View Person Panel.
Timestamp in the status bar is updated.

    1. Test case: `view 0`<br>
       Expected: No person is viewed. Error details shown in the status message. Status bar remains the same.

    1. Other incorrect view commands to try: `view`, `view x`, `...` (where x is larger than the list size)<br>
       Expected: Similar to previous.

1. _{ more test cases …​ }_

### Saving data

1. Dealing with missing/corrupted data files

    1. _{explain how to simulate a missing/corrupted file, and the expected behavior}_

1. _{ more test cases …​ }_

# Appendix

## Planned Enhancements

1. Currently, the view panel can only be updated using the view command. 
However, we are planning on implementing a feature that will allow users to update the view panel by simply clicking on a person in the list panel.
This change is driven by our goal to enhance user experience: although our application primarily caters to CLI users, such  behaviour
still seems intuitive and reasonable to expect.


2. Currently, when the `delete` command encounters invalid indexes, it generates an error and does not delete any patient or specialist records.
   In contrast, the `delsc` command handles invalid shortcuts by recognizing and ignoring them, while continuing to remove any valid shortcuts in the command.
   The inconsistency between these two delete functions has been identified, and we have plans to address it in the future.
   Our upcoming improvement will entail modifying the `delete` command to acknowledge and ignore invalid indexes while effectively deleting records specified by valid indexes provided by the user.
