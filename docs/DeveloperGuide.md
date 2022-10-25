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

:bulb: **Tip:** The `.puml` files used to create diagrams in this document can be found in the [diagrams](https://github.com/se-edu/addressbook-level3/tree/master/docs/diagrams/) folder. Refer to the [_PlantUML Tutorial_ at se-edu/guides](https://se-education.org/guides/tutorials/plantUml.html) to learn how to create and edit diagrams.
</div>

### Architecture

<img src="images/ArchitectureDiagram.png" width="280" />

The ***Architecture Diagram*** given above explains the high-level design of the App.

Given below is a quick overview of main components and how they interact with each other.

**Main components of the architecture**

**`Main`** has two classes called [`Main`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/Main.java) and [`MainApp`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/MainApp.java). It is responsible for,
* At app launch: Initializes the components in the correct sequence, and connects them up with each other.
* At shut down: Shuts down the components and invokes cleanup methods where necessary.

[**`Commons`**](#common-classes) represents a collection of classes used by multiple other components.

The rest of the App consists of four components.

* [**`UI`**](#ui-component): The UI of the App.
* [**`Logic`**](#logic-component): The command executor.
* [**`Model`**](#model-component): Holds the data of the App in memory.
* [**`Storage`**](#storage-component): Reads data from, and writes data to, the hard disk.


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

The UI consists of a `MainWindow` that is made up of parts e.g.`CommandBox`, `ResultDisplay`, `PersonListPanel`, `NoteListPanel`, `PersonInspectPanel` `StatusBarFooter` etc. All these, including the `MainWindow`, inherit from the abstract `UiPart` class which captures the commonalities between classes that represent parts of the visible GUI.

The `UI` component uses the JavaFx UI framework. The layout of these UI parts are defined in matching `.fxml` files that are in the `src/main/resources/view` folder. For example, the layout of the [`MainWindow`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/ui/MainWindow.java) is specified in [`MainWindow.fxml`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/resources/view/MainWindow.fxml)

The `UI` component is separate from the state of the `Model` and changes to the `UI` will not affect the `Model`, but changes to the `Model`'s state will affect the information displayed by the `UI` elements.

Similarly, changes to the visual aspects of `UI`, such as current person viewed, or the filtered state of the lists, will not be saved to file as they do not affect the `Model`'s data.

The `UI` component,

* executes user commands using the `Logic` component.
* listens for changes to `Model` data so that the UI can be updated with the modified data.
* keeps a reference to the `Logic` component, because the `UI` relies on the `Logic` to execute commands.
* depends on some classes in the `Model` component, as it displays `Person` and `Notes` object residing in the `Model`.

#### UI Elements

1. **Command Box**: A text box that allows users to enter in commands for later execution.
2. **ResultDisplay**: A readonly text box that serves as a console to give feedback from the `Logic` component to the user, such as error messages or logs.
3. **Person List**: A horizontal sliding list that displays all persons in the SectresBook. This list can be filtered to display only relevant people according to some predicate.
4. **Note List**: A vertical sliding list that displays all notes in the SectresBook. This list can be filtered to display only relevant notes according to some predicate.
5. **Person Inspect Panel**: This Panel displays data of a person, using the UI command `inspect`.
6. **Status Bar Footer**: This footer displays the address where the data file is saved to.

### Logic component

**API** : [`Logic.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/logic/Logic.java)

Here's a (partial) class diagram of the `Logic` component:

<img src="images/LogicClassDiagram.png" width="550"/>

How the `Logic` component works:
1. When `Logic` is called upon to execute a command, it uses the `AddressBookParser` class to parse the user command.
1. This results in a `Command` object (more precisely, an object of one of its subclasses e.g., `AddCommand`) which is executed by the `LogicManager`.
1. The command can communicate with the `Model` when it is executed (e.g. to add a person).
1. The result of the command execution is encapsulated as a `CommandResult` object which is returned back from `Logic`.

The Sequence Diagram below illustrates the interactions within the `Logic` component for the `execute("delete 1")` API call.

![Interactions Inside the Logic Component for the `delete 1` Command](images/DeleteSequenceDiagram.png)

<div markdown="span" class="alert alert-info">:information_source: **Note:** The lifeline for `DeleteCommandParser` should end at the destroy-marker (X) but due to a limitation of PlantUML, the lifeline reaches the end of diagram.
</div>

Commands such as Edit and Delete feature the ability to delete by name, which utilises the Find feature. Illustrated here is how `execute("edit Lynette")` interacts with the Logic component, using a sequence diagram

![Interactions Inside the Logic Component for the `edit Lynette` Command](images/EditSequenceDiagram.png)

Here are the other classes in `Logic` (omitted from the class diagram above) that are used for parsing a user command:

<img src="images/ParserClasses.png" width="600"/>

How the parsing works:
* When called upon to parse a user command, the `AddressBookParser` class creates an `XYZCommandParser` (`XYZ` is a placeholder for the specific command name e.g., `AddCommandParser`) which uses the other classes shown above to parse the user command and create a `XYZCommand` object (e.g., `AddCommand`) which the `AddressBookParser` returns back as a `Command` object.
* All `XYZCommandParser` classes (e.g., `AddCommandParser`, `DeleteCommandParser`, ...) inherit from the `Parser` interface so that they can be treated similarly where possible e.g, during testing.

### Model component
**API** : [`Model.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/model/Model.java)

<img src="images/ModelClassDiagram.png" width="450" />


The `Model` component,

* stores the address book data i.e., all `Person` objects (which are contained in a `UniquePersonList` object) and all `Note` objects (contained in a `NoteBook` object)
* stores the currently 'selected' `Person` objects (e.g., results of a search query) as a separate _filtered_ list which is exposed to outsiders as an unmodifiable `ObservableList<Person>` that can be 'observed' e.g. the UI can be bound to this list so that the UI automatically updates when the data in the list change.
* stores a `UserPref` object that represents the user’s preferences. This is exposed to the outside as a `ReadOnlyUserPref` objects.
* does not depend on any of the other three components (as the `Model` represents data entities of the domain, they should make sense on their own without depending on other components)




### Storage component

**API** : [`Storage.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/storage/Storage.java)

<img src="images/StorageClassDiagram.png" width="550" />

The `Storage` component,
* can save both address book data and user preference data in json format, and read them back into corresponding objects.
* inherits from both `AddressBookStorage` and `UserPrefStorage`, which means it can be treated as either one (if only the functionality of only one is needed).
* depends on some classes in the `Model` component (because the `Storage` component's job is to save/retrieve objects that belong to the `Model`)

### Common classes

Classes used by multiple components are in the `seedu.addressbook.commons` package.

--------------------------------------------------------------------------------------------------------------------

## **Implementation**

This section describes some noteworthy details on how certain features are implemented.

### addNote feature

#### Implementation

The addNote mechanism is facilitated by `AddNoteCommand`. It extends `Command` and overrides `Command#execute()` to implement the following operation:
- `AddNoteCommand#execute()` : adds the specified note with its associated title and content into the list of notes to be kept track of.

Given below is an example usage scenario and how the addNote mechanism behaves at each step.

Step 1. The user launches the application and wishes to keep track of a note with the following attributes :
1. Title : Club meeting
2. Content : 3rd October 9pm, brief everybody on upcoming events.

Step 2. The user executes `addNote n_t/Meeting n_c/3rd October 9pm`, which calls `LogicManager#execute()`. Subsequently, `AddressBookParser#parseCommand()` is called
which will create a `AddNoteCommandParser` object and call `AddNoteCommandParser#parse()`. This method will take the user's input and make sense of it to create a `Note` object.

Step 3. An `AddNoteCommand` will be created and `AddNoteCommand#execute()` will be called by `LogicManager#execute()`.

Step 4. `AddNoteCommand#execute()` will call the following method from `Model` :
- `addNote(toAdd)`

Step 5. `AddNoteCommand#execute()` will return a `CommandResult` object which will display the following message back to the user:
> New note added: Title: Meeting, Content: 3rd October 9pm

The following sequence diagram shows how the addNote operation works:

![AddNoteSequenceDiagram](images/AddNoteSequenceDiagram.png)

#### Design considerations

**Aspect: How Title and Content are represented:**

* **Alternative 1 (current choice):** Title and Content as separate objects.
    * Pros: Easy to validate Title/Content. (In the respective classes)
    * Cons: May have performance issues in terms of memory usage(Many objects might be created).

* **Alternative 2:** Title and Content as fields of Note
    * Pros: Will use less memory (Fewer objects created).
    * Cons: Harder to validate Title/Content. Better OOP(Object-oriented programming) design.

### deleteNote feature

#### Proposed implementation

The deleteNote mechanism is facilitated by `DeleteNoteCommand`. It extends `Command` and overrides `Command#execute()` to implement the following operation:
- `DeleteNoteCommand#execute()` : deletes the note at the specified index from the note list.

Given below is an example usage scenario and how the addNote mechanism behaves at each step.

Step 1. The user launches the application and wishes to delete a note that no longer needs to be kept track of. The user lists the current notes:
1. Title: Meeting, Content: 3rd October 9pm 
2. Title: Event, Content: Remind club members to attend.

The user has decided to delete note 1.

Step 2. The user executes `deleteNote 1`, which calls `LogicManager#execute()`. Subsequently, `AddressBookParser#parseCommand()` is called
which will create a `DeleteNoteCommandParser` object and call `DeleteNoteCommandParser#parse()`. This method will take the user's input and make sense of it to get the index of note to be deleted.

Step 3. A `DeleteNoteCommand` object will be created and `DeleteNoteCommand#execute()` will be called by `LogicManager#execute()`.

Step 4. `DeleteNoteCommand#execute()` will call the following method from `Model` :
- `getAddressBook()`
- `deleteNote(noteToDelete)`

Step 5. `DeleteNoteCommand#execute()` will return a `CommandResult` object which will display the following message back to the user:
> Deleted Note: Title: Meeting, Content: 3rd October 9pm

The following sequence diagram shows how the addNote operation works:

![DeleteNoteSequenceDiagram](images/DeleteNoteSequenceDiagram.png)

#### Design considerations

**Aspect: How the note to be deleted is specified:**

* **Alternative 1 (current choice):** Note is specified by index.
    * Pros: Easy to implement.
    * Cons: Would need to use listNote command or gui to allow easy identification of index of note.

* **Alternative 2:** Note is specified by Title.
    * Pros: Would be more precise (Title of notes are unique).
    * Cons: Long command would be needed to delete a note with a long Title.

### Find Person

### Implementation

The Find Person feature is facilitated by 'FindCommand'. It allows users to find all Persons with names that are matching or phone number starting with any of the keywords.

Given below is an example usage scenario and how the find feature behaves at each step.

Step 1. The user executes 'find David' command to find all Persons in the address book that includes the name `David`.

Step 2. A `FindCommand` is constructed with a `NameContainsKeywordPredicate` which checks through the list of persons in the address book and only shows those with their first/last name matching `David`.

Step 3. The `FindCommand` is executed and the `NameContainsKeywordsPredicate` is used to update the filtered person list.

The following sequence diagram shows how the find command works:

<img src="images/FindCommandSequence.png" width="740"/>

<div markdown="span" class="alert alert-info">:information_source: **Note:** The lifeline for `FindCommandParser` and `FindCommand` should end at the destroy marker (X) but due to a limitation of PlantUML, the lifeline reaches the end of diagram.

#### Design considerations:

**Aspect: How find executes:**

* **Alternative 1 (current choice):** Chooses person based on the whole first/last name matching the keyword.
    * Pros: More specific Persons list after the find command.
    * Cons: User needs to know the first/last name of the person they are trying to find.

* **Alternative 2:** Chooses person if name contains the keyword.
    * Pros: Easier to find person.
    * Cons: Persons list may show other persons that are not desired by the user.

### Find Person by Tag feature

#### Implementation

The find Person by Tag feature (called `findTag`) is facilitated by `FindTagCommand`. It allows users to find all Persons with the given Tags.

Given below is an example usage scenario and how the findTag feature behaves at each step.

Step 1. The user executes `findTag Finance` command to find all Persons in the address book with the tag `Finance`.

Step 2. A `FindTagCommand` is constructed with a `TagsContainsKeywordsPredicate` which will check through the list of persons in the address book and only show those with the tag `Finance`.

Step 3. The `FindTagCommand` is executed and the `TagsContainsKeywordsPredicate` is passed to model to update the Person List to only show Persons with the tag `Finance`.

The following sequence diagram shows how the findTag command works:

<img src="images/FindTagSequenceDiagram.png" width="740"/>

<div markdown="span" class="alert alert-info">:information_source: **Note:** The lifeline for `FindTagCommandParser` and `FindTagCommand` should end at the destroy marker (X) but due to a limitation of PlantUML, the lifeline reaches the end of diagram.

#### Design considerations:

**Aspect: How findTag executes:**

* **Alternative 1 (current choice):** Goes through all Persons to check for Tag.
    * Pros: Easy to implement (Similar to current find command).
    * Cons: May have performance issues in terms of having to do many more steps.

* **Alternative 2:** Goto searched Tags and get the Persons that each Tag points to.
    * Pros: Will use fewer steps (Go directly to the Tags rather than looking through all Persons).
    * Cons: Implementation would be more complicated.

### \[Proposed\] Undo/redo feature

#### Proposed Implementation

The proposed undo/redo mechanism is facilitated by `VersionedAddressBook`. It extends `AddressBook` with an undo/redo history, stored internally as an `addressBookStateList` and `currentStatePointer`. Additionally, it implements the following operations:

* `VersionedAddressBook#commit()` — Saves the current address book state in its history.
* `VersionedAddressBook#undo()` — Restores the previous address book state from its history.
* `VersionedAddressBook#redo()` — Restores a previously undone address book state from its history.

These operations are exposed in the `Model` interface as `Model#commitAddressBook()`, `Model#undoAddressBook()` and `Model#redoAddressBook()` respectively.

Given below is an example usage scenario and how the undo/redo mechanism behaves at each step.

Step 1. The user launches the application for the first time. The `VersionedAddressBook` will be initialized with the initial address book state, and the `currentStatePointer` pointing to that single address book state.

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

<div markdown="span" class="alert alert-info">:information_source: **Note:** The lifeline for `UndoCommand` should end at the destroy marker (X) but due to a limitation of PlantUML, the lifeline reaches the end of diagram.

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

#### Design considerations:

**Aspect: How undo & redo executes:**

* **Alternative 1 (current choice):** Saves the entire address book.
  * Pros: Easy to implement.
  * Cons: May have performance issues in terms of memory usage.

* **Alternative 2:** Individual command knows how to undo/redo by
  itself.
  * Pros: Will use less memory (e.g. for `delete`, just save the person being deleted).
  * Cons: We must ensure that the implementation of each individual command are correct.

_{more aspects and alternatives to be added}_

### \[Proposed\] Data archiving

_{Explain here how the data archiving feature will be implemented}_


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

* is acting as a secretary or a treasurer of a club with a lot of people
* has a need for a convenient way to organise paperwork and general information about the club
* has a need to manage a significant number of contacts
* has a requirement to keep notes and tabs on people and projects
* prefer desktop apps over other types
* can type fast
* prefers typing to mouse interactions
* is reasonably comfortable using CLI apps

**Value proposition**: manage club members and track notes faster than a typical mouse/GUI driven app or by pen and paper


### User stories

Priorities: High (must have) - `* * *`, Medium (nice to have) - `* *`, Low (unlikely to have) - `*`

| Priority | As a …​      | I want to …​                                             | So that I can…​                                                        |
| -------- |--------------|----------------------------------------------------------|------------------------------------------------------------------------|
| `* * *`  | secretary    | add club members’ information into the address book      | keep track of their contact information.                               |
| `* * *`  | secretary    | edit a club member’s information                         | stay updated with them if their contact information changes.           |
| `* * *`  | secretary    | delete a club member’s information from the address book | stop keeping track of them when they leave the club.                   |
| `* * *`  | user         | search for a person by their name or contact number      | locate details of persons without having to go through the entire list |
| `* *`    | secretary    | search contacts according to a specific tag              | easily  contact people in a whole group                                |
| `*`      | user         | maintain a set of tasks to be done                       | keep track of things to be done.                                       |

*{More to be added}*

### Use cases

(For all use cases below, the **System** is the `SectresBook` and the **Actor** is the `user`, unless specified otherwise)

**Use case: Add a person**

**MSS**
1. User requests to add a person.
2. SectresBook adds the person to the list of persons.

    Use case ends.

**Extensions**
* 1a. The given person already exists.
  * 1a1. SectresBook shows an error message.

    Use case ends.
* 1b. Necessary fields are incomplete/empty.
  * 1b1. Sectresbook shows an error message.

    Use case ends.

**Use case: Update a person**

**MSS**
1. User requests to list persons.
2. SectresBook shows a list of persons.
3. User requests to update a specific person in the list.
4. SectresBook updates information of the person.

    Use case ends.

**Extensions**
* 2a. The list is empty.

    Use case ends.
* 3a. The given index is invalid.
  * 3a1. SectresBook shows an error message.

    Use case resumes at step 2.
* 3b. The command line arguments are invalid.
  * 3b1. SectresBook shows an error message.

    Use case resumes at step 2.

**Use case: Delete a person**

**MSS**

1.  User requests to list persons.
2.  SectresBook shows a list of persons.
3.  User requests to delete a specific person in the list.
4.  SectresBook deletes the person.

    Use case ends.

**Extensions**

* 2a. The list is empty.

  Use case ends.

* 3a. The given index is invalid.

    * 3a1. SectresBook shows an error message.

      Use case resumes at step 2.

**Use case: Find a person**

**MSS**
1. User request to find using keyword.
2. SectressBook shows a list of persons matching keyword.

    Use case ends.

**Use case: Display list of persons**

**MSS**
1. User requests to list persons.
2. SectresBook displays the list of persons stored.

    Use case ends.

*{More to be added}*

### Non-Functional Requirements

1. Should work on any _mainstream OS_ as long as it has Java `11` or above installed.
2. Should be able to hold up to 1000 persons without a noticeable sluggishness in performance for typical usage.
3. A user with above average typing speed for regular English text (i.e. not code, not system admin commands) should be able to accomplish most of the tasks faster using commands than using the mouse.
4. Should not require internet connection, all operations are performed locally.
5. Should not consume a lot of battery to keep it running in the background

### Glossary

* **Mainstream OS**: Windows, Linux, Unix, OS-X
* **Private contact detail**: A contact detail that is not meant to be shared with others
* **Note**: A segment of text that describes a task to be done, coupled with tags that reference people in the SectresBook who are associated with the given task.
* **Secretary**: A person acting as overseers for the administrative functions of a club.

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

### Saving data

1. Dealing with missing/corrupted data files

   1. _{explain how to simulate a missing/corrupted file, and the expected behavior}_

1. _{ more test cases …​ }_
