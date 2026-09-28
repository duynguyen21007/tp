---
  layout: default.md
  title: "Developer Guide"
  pageNav: 3
---

# Laplace Developer Guide

<!-- * Table of Contents -->
<page-nav-print />

--------------------------------------------------------------------------------------------------------------------

## **Acknowledgements**

* This project evolves [AddressBook Level 3](https://github.com/se-edu/addressbook-level3). The architecture, implementation examples, and initial manual tests below are adapted from its Developer Guide. They describe the inherited codebase until the corresponding Laplace features replace them.

--------------------------------------------------------------------------------------------------------------------

## **Setting up, getting started**

Refer to the guide [_Setting up and getting started_](SettingUp.md).

--------------------------------------------------------------------------------------------------------------------

## **Design**

This section documents the inherited AddressBook Level 3 implementation, which is still the basis of the current code. The Laplace requirements in the appendix describe the intended product; diagrams and component descriptions here must be updated as that implementation changes.

### Architecture

<puml src="diagrams/ArchitectureDiagram.puml" width="280" />

The ***Architecture Diagram*** given above explains the high-level design of the App.

The following provides a quick overview of the main components and their interactions.

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

<puml src="diagrams/ArchitectureSequenceDiagram.puml" width="574" />

Each of the four main components (also shown in the diagram above),

* defines its *API* in an `interface` with the same name as the Component.
* provides its functionality through a concrete `{Component Name}Manager` class that implements the corresponding API interface.

For example, the `Logic` component defines its API in `Logic.java` and implements it in `LogicManager.java`. Other components interact with a component through its interface rather than its concrete class, preventing them from coupling to that component's implementation, as illustrated in the following partial class diagram.

<puml src="diagrams/ComponentManagers.puml" width="300" />

The sections below give more details of each component.

### UI component

The **API** of this component is specified in [`Ui.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/ui/Ui.java)

<puml src="diagrams/UiClassDiagram.puml" alt="Structure of the UI Component"/>

The UI consists of a `MainWindow` and its parts, such as `CommandBox`, `ResultDisplay`, `PersonListPanel`, and `StatusBarFooter`. All of these, including `MainWindow`, inherit from the abstract `UiPart` class, which captures common behavior among classes that represent visible GUI parts.

The `UI` component uses the JavaFX UI framework. The layouts of these UI parts are defined in matching `.fxml` files in `src/main/resources/view`. For example, [`MainWindow.fxml`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/resources/view/MainWindow.fxml) specifies the layout of [`MainWindow`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/ui/MainWindow.java).

The `UI` component,

* executes user commands using the `Logic` component.
* listens for changes to `Model` data so that the UI can be updated with the modified data.
* keeps a reference to the `Logic` component, because the `UI` relies on the `Logic` to execute commands.
* depends on some classes in the `Model` component because it displays `Person` objects from the model.

### Logic component

**API** : [`Logic.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/logic/Logic.java)

Here's a (partial) class diagram of the `Logic` component:

<puml src="diagrams/LogicClassDiagram.puml" width="550"/>

The sequence diagram below illustrates the interactions within the `Logic` component, taking `execute("delete 1")` API call as an example.

<puml src="diagrams/DeleteSequenceDiagram.puml" alt="Interactions Inside the Logic Component for the `delete 1` Command" />

<box type="info" seamless>

**Note:** The lifeline for `DeleteCommandParser` should end at the destroy marker (X), but due to a limitation of PlantUML, the lifeline continues till the end of diagram.
</box>


How the `Logic` component works:

1. When `Logic` is called upon to execute a command, the command is passed to an `AddressBookParser` object, which in turn creates a parser that matches the command (e.g., `DeleteCommandParser`) and uses it to parse the command.
1. This results in a `Command` object (more precisely, an object of one of its subclasses e.g., `DeleteCommand`) which is executed by the `LogicManager`.
1. The command can communicate with the `Model` when it is executed (e.g. to delete a person).<br>
   Note that although this is shown as a single step in the diagram above for simplicity, the code can require several interactions between the command object and the `Model` to complete the operation.
1. The result of the command execution is encapsulated as a `CommandResult` object which is returned from `Logic`.

Here are the other classes in `Logic` (omitted from the class diagram above) that are used for parsing a user command:

<puml src="diagrams/ParserClasses.puml" width="600"/>

How the parsing works:
* When called upon to parse a user command, the `AddressBookParser` class creates an `XYZCommandParser` (`XYZ` is a placeholder for the specific command name, e.g., `AddCommandParser`). The parser uses the other classes shown above to parse the user command and create an `XYZCommand` object (e.g., `AddCommand`). The `AddressBookParser` returns that object as a `Command` object.
* All `XYZCommandParser` classes, such as `AddCommandParser` and `DeleteCommandParser`, implement the `Parser` interface so they can be treated similarly where appropriate, for example during testing.

### Model component
**API** : [`Model.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/model/Model.java)

<puml src="diagrams/ModelClassDiagram.puml" width="450" />


The `Model` component,

* stores the address book data i.e., all `Person` objects (which are contained in a `UniquePersonList` object).
* stores the `Person` objects selected by the current filter, such as search results, in a separate _filtered_ list. It exposes this list as an unmodifiable `ObservableList<Person>` that the UI can observe and bind to, so the UI updates when the list changes.
* stores a `UserPrefs` object that represents the user’s preferences (currently, just the GUI settings). This is exposed to the outside as a `ReadOnlyUserPrefs` object.
* does not depend on any of the other three components (as the `Model` represents data entities of the domain, they should make sense on their own without depending on other components)


<box type="info" seamless>

**Note:** The alternative, arguably more object-oriented, design below keeps a unique list of tags in `AddressBook`, and each `Person` references tags from that list. This lets `AddressBook` maintain one `Tag` object per unique tag instead of each `Person` holding its own `Tag` objects.<br>

<puml src="diagrams/BetterModelClassDiagram.puml" width="450" />
</box>


### Storage component

**API** : [`Storage.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/storage/Storage.java)

<puml src="diagrams/StorageClassDiagram.puml" width="550" />

The `Storage` component,
* can save both address book data and user preference data in JSON format, and read them back into corresponding objects.
* is implemented by `StorageManager`, which delegates the actual JSON file access to `JsonAddressBookStorage` and `JsonUserPrefsStorage` (one class per data file).
* depends on some classes in the `Model` component (because the `Storage` component's job is to save/retrieve objects that belong to the `Model`)

### Common classes

Classes used by multiple components are in the `seedu.address.commons` package.

--------------------------------------------------------------------------------------------------------------------

## **Implementation**

This section retains proposed work from the inherited AddressBook Level 3 guide. Its `Person` and `AddressBook` examples do not describe implemented Laplace member, equipment, or loan behavior.

### \[Proposed\] Undo/redo feature

#### Proposed Implementation

The proposed undo/redo mechanism is facilitated by `VersionedAddressBook`. It extends `AddressBook` with an undo/redo history, stored internally as an `addressBookStateList` and `currentStatePointer`. Additionally, it implements the following operations:

* `VersionedAddressBook#commit()` -- Saves the current address book state in its history.
* `VersionedAddressBook#undo()` -- Restores the previous address book state from its history.
* `VersionedAddressBook#redo()` -- Restores a previously undone address book state from its history.

These operations are exposed in the `Model` interface as `Model#commitAddressBook()`, `Model#undoAddressBook()` and `Model#redoAddressBook()` respectively.

Given below is an example usage scenario and how the undo/redo mechanism behaves at each step.

Step 1. The user launches the application for the first time. The `VersionedAddressBook` will be initialized with the initial address book state, and the `currentStatePointer` pointing to that single address book state.

<puml src="diagrams/UndoRedoState0.puml" alt="UndoRedoState0" />

Step 2. The user executes `delete 5` command to delete the 5th person in the address book. The `delete` command calls `Model#commitAddressBook()`, causing the modified state of the address book after the `delete 5` command executes to be saved in the `addressBookStateList`, and the `currentStatePointer` is shifted to the newly inserted address book state.

<puml src="diagrams/UndoRedoState1.puml" alt="UndoRedoState1" />

Step 3. The user executes `add n/David …​` to add a new person. The `add` command also calls `Model#commitAddressBook()`, causing another modified address book state to be saved into the `addressBookStateList`.

<puml src="diagrams/UndoRedoState2.puml" alt="UndoRedoState2" />

<box type="info" seamless>

**Note:** If a command fails its execution, it will not call `Model#commitAddressBook()`, so the address book state will not be saved into the `addressBookStateList`.
</box>

Step 4. The user now decides that adding the person was a mistake, and decides to undo that action by executing the `undo` command. The `undo` command will call `Model#undoAddressBook()`, which will shift the `currentStatePointer` once to the left, pointing it to the previous address book state, and restores the address book to that state.

<puml src="diagrams/UndoRedoState3.puml" alt="UndoRedoState3" />


<box type="info" seamless>

**Note:** If the `currentStatePointer` is at index 0, pointing to the initial AddressBook state, then there are no previous AddressBook states to restore. The `undo` command uses `Model#canUndoAddressBook()` to check if this is the case. If so, it will return an error to the user rather
than attempting to perform the undo.
</box>

The following sequence diagram shows how an undo operation goes through the `Logic` component:

<puml src="diagrams/UndoSequenceDiagram-Logic.puml" alt="UndoSequenceDiagram-Logic" />

<box type="info" seamless>

**Note:** The lifeline for `UndoCommand` should end at the destroy marker (X), but due to a limitation of PlantUML, it continues to the end of the diagram.
</box>

Similarly, how an undo operation goes through the `Model` component is shown below:

<puml src="diagrams/UndoSequenceDiagram-Model.puml" alt="UndoSequenceDiagram-Model" />

The `redo` command does the opposite — it calls `Model#redoAddressBook()`, which shifts the `currentStatePointer` once to the right, pointing to the previously undone state, and restores the address book to that state.

<box type="info" seamless>

**Note:** If the `currentStatePointer` is at index `addressBookStateList.size() - 1`, pointing to the latest address book state, then there are no undone AddressBook states to restore. The `redo` command uses `Model#canRedoAddressBook()` to check if this is the case. If so, it will return an error to the user rather than attempting to perform the redo.
</box>

Step 5. The user then decides to execute the command `list`. Commands that do not modify the address book, such as `list`, will usually not call `Model#commitAddressBook()`, `Model#undoAddressBook()` or `Model#redoAddressBook()`. Thus, the `addressBookStateList` remains unchanged.

<puml src="diagrams/UndoRedoState4.puml" alt="UndoRedoState4" />

Step 6. The user executes `clear`, which calls `Model#commitAddressBook()`. Since the `currentStatePointer` is not pointing at the end of the `addressBookStateList`, all address book states after the `currentStatePointer` will be purged. Reason: It no longer makes sense to redo the `add n/David …` command. This is the behavior that most modern desktop applications follow.

<puml src="diagrams/UndoRedoState5.puml" alt="UndoRedoState5" />

The following activity diagram summarizes what happens when a user executes a new command:

<puml src="diagrams/CommitActivityDiagram.puml" width="250" />

#### Design considerations:

**Aspect: How undo & redo execute:**

* **Alternative 1 (current choice):** Saves the entire address book.
    * Pros: Easy to implement.
    * Cons: May have performance issues in terms of memory usage.

* **Alternative 2:** Individual command knows how to undo/redo by
  itself.
    * Pros: Will use less memory (e.g. for `delete`, just save the person being deleted).
    * Cons: We must ensure that the implementation of each individual command is correct.

### \[Proposed\] Data archiving

Laplace would retain an archived member's record and closed loans while excluding that member from normal active lists. Archiving must reject members with open loans. The storage format would need an archive state, and member lookup would need to distinguish active from archived records. This is a proposal; the inherited AddressBook Level 3 model has no member archive state.


--------------------------------------------------------------------------------------------------------------------

## **Documentation, logging, testing, dev-ops**

* [Documentation guide](Documentation.md)
* [Testing guide](Testing.md)
* [Logging guide](Logging.md)
* [DevOps guide](DevOps.md)

--------------------------------------------------------------------------------------------------------------------

## **Appendix: Requirements**

### Product scope

Laplace is a local, single-user desktop application for a CCA EXCO member to manage the CCA's members, equipment inventory, and equipment loans.

**Target user profile**:

* Is a CCA EXCO member responsible for maintaining member records and tracking equipment issued to members.
* Needs to distinguish individual equipment units, identify their current holders, and record issue and return dates.
* Maintains the records on their own computer; members are recorded entities, not additional application users.
* Can type quickly, prefers commands to repeated mouse interactions, and is comfortable learning a small command vocabulary.
* Wants a consolidated view of membership, inventory, and equipment responsibility during day-to-day club operations.

**Value proposition**: Laplace helps a CCA EXCO member keep member details, equipment records, and loans in one place. Commands support quick record entry and lookup, while linked member and equipment views show who holds each item. Availability checks prevent double assignments, reducing confusion and manual cross-checking during equipment issue and return.

**Requirements baseline and scope**:

These requirements consolidate the [team's project notes](https://docs.google.com/document/d/1XzYR2IpGc__mSZEy5GKMmVK8809j3Ln42VA8_9-va48/edit?usp=sharing): Week 4 (direction), Week 5 (all 41 user stories), and Week 6 (MVP and feature specifications). They describe intended behavior, including requirements beyond the MVP; they do not claim that every feature is implemented. Where the earlier schema differs from the later Feature Specifications, the latter defines the MVP baseline.

* **MVP baseline:** Add, list, view, and delete members and equipment; identify members by NUS ID and equipment by a generated UUID; issue and return individual items; show holdings and holders; validate input and preserve consistency on failure.
* **Broader requirements:** Editing, archiving, recovery, import/export, search and filtering, summaries, batch operations, condition recording at handover, histories, duplicate warnings, and undo/redo remain documented below even if they are not delivered this semester. Batch addition and per-type summaries require an equipment type field shared by physical units; the MVP equipment schema does not contain that field.
* **Deletion and retention:** The MVP permanently deletes an eligible record and its closed loans. The proposed 30-day recovery and historical lookup requirements need a revised retention policy before implementation: recoverable records and their linked loans must be retained for recovery, and archived members must retain their history. The MVP deletion behavior does not satisfy these future requirements.
* **Single-user boundary:** Import/export supports the same user's local records and backups. The earlier idea of handing a snapshot to another EXCO member is retained as a considered use of export, but is excluded from the course product's regular operations. Shared datasets, concurrent users, and member self-service are outside scope.

### User stories

Priorities: High (must have) — `* * *`; Medium (nice to have) — `* *`; Low (lower-priority candidate) — `*`.
Priority expresses product importance, not implementation status. **MVP** denotes the Week 6 baseline; **Future** denotes a requirement outside that baseline, without a delivery commitment. US01–US41 retain the numbering of the Week 5 stories; US42–US45 make supporting requirements explicit. US46 preserves the broader apparent-duplicate warning in the original US13; the MVP only rejects identical NUS IDs.

| ID | Priority | Scope | As a … | I want to … | So that I can … |
|----|----------|-------|--------|-------------|----------------|
| US01 | `* *` | Future | newly appointed CCA EXCO member | import member and equipment records from a local file | avoid re-entering existing lists manually. |
| US02 | `* *` | Future | cautious CCA EXCO member | preview additions, changes, and rejected records before confirming an import | correct problems before changing my current records. |
| US03 | `* *` | Future | CCA EXCO member | export all Laplace data to one portable snapshot file | keep a complete backup of my records. |
| US04 | `* * *` | MVP | CCA EXCO member | add a member with their NUS ID, name, phone number, and email | keep track of a person who has joined the CCA. |
| US05 | `* * *` | MVP | CCA EXCO member | list all active members | review the current membership at a glance. |
| US06 | `* * *` | MVP | CCA EXCO member | view one member's full details | verify their information before an operation. |
| US07 | `* *` | Future | CCA EXCO member | edit a member's details while keeping their identifier stable | keep corrected or changed information accurate without breaking loan references. |
| US08 | `* *` | Future | CCA EXCO member | archive an inactive member | keep the active list relevant while retaining historical records. |
| US09 | `* * *` | MVP | CCA EXCO member | delete a member record created in error when it has no outstanding equipment | remove unwanted records without losing track of a current holder. |
| US10 | `* *` | Future | CCA EXCO member | restore a deleted member within 30 days of deletion | recover from an accidental deletion. |
| US11 | `* *` | Future | CCA EXCO member | filter and sort members by a chosen field | focus on the people relevant to my current task. |
| US12 | `* *` | Future | busy CCA EXCO member | find members using incomplete queries or approximate names | retrieve the right record quickly even when I do not know the exact identifier. |
| US13 | `* * *` | MVP | CCA EXCO member adding data | have duplicate NUS IDs rejected with an explanation | avoid conflicting records for the same member. |
| US14 | `* * *` | MVP | CCA EXCO member | add one equipment item with an automatically generated unique UUID | track each newly acquired physical unit separately. |
| US15 | `*` | Future | CCA EXCO member receiving a batch | add several units of the same equipment type in one operation | record a delivery without repeating the same entry process. |
| US16 | `* * *` | MVP | CCA EXCO member | list all active equipment and its availability | review the current inventory at a glance. |
| US17 | `* * *` | MVP | CCA EXCO member | view one equipment item's full details | verify its identity, condition, and recorded information. |
| US18 | `* *` | Future | CCA EXCO member | edit an equipment record while retaining its UUID | keep item information accurate without breaking loan references. |
| US19 | `* * *` | MVP | CCA EXCO member | delete an equipment record created in error when it is not on loan | remove unwanted entries without losing track of an issued item. |
| US20 | `* *` | Future | CCA EXCO member | restore a deleted equipment record within 30 days of deletion | recover from an accidental deletion. |
| US21 | `* *` | Future | CCA EXCO member | filter and sort equipment by category, condition, or availability | identify suitable items for my current task. |
| US22 | `* *` | Future | busy CCA EXCO member | find equipment using incomplete queries or approximate names | locate an item even when I do not know its UUID. |
| US23 | `* *` | Future | CCA EXCO member planning an activity | see counts of available, assigned, and unavailable units of an equipment type | decide whether there is enough equipment without counting manually. |
| US24 | `* * *` | MVP | CCA EXCO member issuing equipment | assign an available item to a member and record its assignment date | track who is responsible for the item and when it was issued. |
| US25 | `* * *` | MVP | CCA EXCO member issuing equipment | record an expected return date on or after the assignment date | know when the item should come back. |
| US26 | `* * *` | MVP | CCA EXCO member receiving equipment | record an assigned item as returned with its actual return date | keep the holder and availability information accurate. |
| US27 | `* * *` | MVP | CCA EXCO member | view all equipment currently held by one member | check that member's outstanding responsibilities in one place. |
| US28 | `* * *` | MVP | CCA EXCO member | see an assigned item's current holder and contact details | identify whom to approach when the item is needed. |
| US29 | `* * *` | MVP | CCA EXCO member issuing equipment | have assignments of already assigned, damaged, or under-repair items rejected | avoid recording multiple holders or issuing unsuitable equipment. |
| US30 | `*` | Future | CCA EXCO member distributing a kit | assign several items to one member in one operation | process a bundled issue quickly. |
| US31 | `*` | Future | CCA EXCO member receiving a kit | record several items from one member as returned in one operation | process a bundled return quickly. |
| US32 | `* *` | Future | CCA EXCO member issuing equipment | record an item's condition at issue | establish its starting condition for later comparison. |
| US33 | `* *` | Future | CCA EXCO member receiving equipment | record an item's condition at return | identify changes while the handover is still fresh. |
| US34 | `* *` | Future | CCA EXCO member investigating an item | view its past equipment loans | trace who held it and when. |
| US35 | `* *` | Future | CCA EXCO member reviewing a member | view that member's past equipment loans | resolve questions about previous loans without searching separate records. |
| US36 | `* *` | Future | CCA EXCO member archiving a member | be warned and prevented from archiving a member with outstanding equipment | resolve responsibility before removing the member from active records. |
| US37 | `* *` | Future | CCA EXCO member who made a mistake | undo my most recent successful data-changing action | recover without reconstructing the previous state manually. |
| US38 | `* *` | Future | CCA EXCO member | redo an action I just undid | recover when the undo itself was a mistake. |
| US39 | `*` | Future | CCA EXCO member investigating a discrepancy | view a recent history of data-changing actions | understand how the current records came about. |
| US40 | `* *` | Future | CCA EXCO member recovering from data loss | restore a previously exported complete snapshot | resume operations without rebuilding every record. |
| US41 | `* * *` | MVP | CCA EXCO member entering invalid data | receive an actionable error with no partial change applied | correct my input while trusting that the records remain consistent. |
| US42 | `* *` | Future | newly appointed CCA EXCO member | access command instructions and examples | learn operations and check syntax when I forget it. |
| US43 | `* * *` | MVP | CCA EXCO member | retrieve a member by their exact NUS ID | distinguish members even when they share a name. |
| US44 | `* * *` | MVP | CCA EXCO member | retrieve equipment by its exact UUID | distinguish physical units even when their descriptions match. |
| US45 | `* * *` | MVP | CCA EXCO member | have successful changes saved locally and loaded on the next launch | continue managing records across sessions without re-entering them. |
| US46 | `* *` | Future | CCA EXCO member adding data | be warned when a new member appears to duplicate an existing member despite having a different NUS ID | review a possible duplicate without wrongly blocking people who share a name or contact detail. |

### Use cases

For all use cases, the **system** is Laplace and the **actor** is the CCA EXCO member operating their local installation. The application is running with a valid dataset unless stated otherwise. The member borrowing equipment is not a second system actor. **MSS** means main success scenario. Extension labels refer to the MSS step at which the alternative occurs; each extension states where execution resumes or ends.

For every data-changing use case, a validation failure leaves all records unchanged. If saving fails at the final update step, Laplace reports that the operation was not saved and retains the preceding consistent state; the use case ends unsuccessfully. Success is reported only after persistence succeeds. These rules also apply to proposed future operations.

#### UC01: Register a member

**Scope:** MVP. **Related stories:** US04, US13, US41, US45.

**MSS**

1. The EXCO member provides the new member's NUS ID, name, phone number, and email.
2. Laplace validates the details and checks that the normalized NUS ID is not already registered.
3. Laplace saves the member and shows the stored details.

Use case ends.

**Extensions**

* 2a. Required details are missing or invalid.
    * 2a1. Laplace identifies the invalid input and explains the accepted format.
    * Use case resumes at step 1.
* 2b. The NUS ID already exists, ignoring letter case.
    * 2b1. Laplace rejects the duplicate and identifies the existing member.
    * Use case resumes at step 1.

Names, phone numbers, or email addresses shared by different NUS IDs do not by themselves constitute duplicates. For the future US46 warning, a matching normalized name together with a matching phone number or email address is a possible duplicate to review, but it does not block registration.

#### UC02: Register an equipment item

**Scope:** MVP. **Related stories:** US14, US17, US41, US45.

**MSS**

1. The EXCO member provides the item's name, category, condition, and optional notes.
2. Laplace validates the details and generates a UUID that is not already in use.
3. Laplace saves the item and shows its UUID, details, and derived availability.

Use case ends.

**Extensions**

* 2a. Required details are missing or invalid.
    * 2a1. Laplace explains what needs correcting.
    * Use case resumes at step 1.
* 2b. Laplace cannot generate a unique identifier.
    * 2b1. Laplace reports the failure without adding an item.
    * Use case ends.

Matching descriptions are permitted: each physical unit receives its own UUID. An item in `DAMAGED` or `UNDER_REPAIR` condition may be registered but is unavailable for issue.

#### UC03: Issue equipment to a member

**Scope:** MVP. **Related stories:** US06, US16, US24, US25, US27–US29, US43, US44.

**MSS**

1. The EXCO member requests the intended member's details by NUS ID and the equipment inventory.
2. Laplace shows the member's details and current holdings, and the equipment with its availability and UUIDs.
3. The EXCO member requests assignment of one item to that member, supplying the assignment and expected return dates.
4. Laplace validates the references, availability, and dates, then saves an open loan and shows the updated holder and holdings.

Use case ends.

**Extensions**

* 2a. The member identifier is invalid or no matching member exists.
    * 2a1. Laplace explains the problem.
    * Use case resumes at step 1.
* 2b. No equipment is available for issue.
    * 2b1. Laplace shows that no suitable item is available.
    * Use case ends.
* 4a. A supplied identifier is invalid or does not identify an existing record.
    * 4a1. Laplace identifies the invalid or missing reference.
    * Use case resumes at step 3.
* 4b. The item already has an open loan, or its condition is `DAMAGED` or `UNDER_REPAIR`.
    * 4b1. Laplace refuses the assignment and gives the reason, including the current holder when applicable.
    * Use case resumes at step 3.
* 4c. A date is invalid, the assignment date is after today, or the expected return date precedes assignment.
    * 4c1. Laplace explains the violated date rule.
    * Use case resumes at step 3.

#### UC04: Record an equipment return

**Scope:** MVP. **Related stories:** US17, US26–US28, US41, US44.

**MSS**

1. The EXCO member requests an item's details by UUID.
2. Laplace shows the item, current holder, assignment date, and expected return date.
3. The EXCO member requests that the item be recorded as returned and supplies the actual return date.
4. Laplace validates and saves the return, closes the loan, removes the current holder, and updates availability and the member's holdings.

Use case ends.

**Extensions**

* 2a. The UUID is invalid or no item matches it.
    * 2a1. Laplace explains the problem.
    * Use case resumes at step 1.
* 2b. The item has no open loan.
    * 2b1. Laplace reports that the item is not currently assigned.
    * Use case ends.
* 4a. The supplied item reference is invalid, missing, or has no open loan, including an already recorded return.
    * 4a1. Laplace rejects the request and explains the problem.
    * Use case resumes at step 1.
* 4b. The actual return date is invalid, before assignment, or after today.
    * 4b1. Laplace explains the allowed date range.
    * Use case resumes at step 3.

Passing the expected return date never closes a loan automatically. A return after the expected date is valid. After return, only items in `GOOD` or `FAIR` condition become available.

#### UC05: Delete an erroneous member or equipment record

**Scope:** MVP. **Related stories:** US05, US09, US16, US19, US41.

**MSS**

1. The EXCO member requests the member list or equipment list, according to the kind of record to remove.
2. Laplace shows the requested records and their identifiers.
3. The EXCO member requests deletion of one record using its NUS ID or UUID, respectively.
4. Laplace verifies that the record has no open loans, permanently deletes it and its associated closed loans, and confirms the deletion.

Use case ends.

**Extensions**

* 2a. The requested list is empty.
    * Use case ends.
* 4a. The identifier is invalid or no matching record exists.
    * 4a1. Laplace reports the problem without deleting anything.
    * Use case resumes at step 3.
* 4b. The member still holds equipment, or the equipment item is currently assigned.
    * 4b1. Laplace refuses deletion and explains that the outstanding items must first be returned.
    * Use case ends without deletion. The EXCO member may complete UC04 and start UC05 again.

This is the MVP deletion policy. The future recovery and history stories require the retention changes described under Product scope.

#### UC06: Import records after reviewing a preview

**Scope:** Future. **Related stories:** US01, US02, US13, US41.

**MSS**

1. The EXCO member requests import from a local member and equipment data file.
2. Laplace reads and validates the file, showing a preview of records to add, change, or reject without altering current data.
3. The EXCO member reviews a preview with no unresolved errors and confirms the import.
4. Laplace applies and saves the previewed changes as one operation and reports the result.

Use case ends.

**Extensions**

* 2a. The file is missing, unreadable, or in an unsupported format.
    * 2a1. Laplace explains why it cannot produce a preview.
    * Use case resumes at step 1.
* 2b. Records are invalid, identifiers conflict, or the proposed changes would break loan references.
    * 2b1. Laplace identifies the affected records and reasons and prevents confirmation until they are resolved.
    * Use case resumes at step 1 after the EXCO member corrects or chooses another file.
* 3a. The EXCO member cancels.
    * Use case ends with current data unchanged.
* 4a. The source file or current dataset has changed since the preview.
    * 4a1. Laplace invalidates the old preview without applying it.
    * Use case resumes at step 2.

#### UC07: Restore a complete backup

**Scope:** Future. **Related stories:** US03, US40, US41.
**Precondition:** The EXCO member has a previously exported complete snapshot. Laplace can accept a restore request even if its normal data file cannot be loaded.

**MSS**

1. The EXCO member selects a local snapshot for restoration.
2. Laplace validates the complete snapshot and explains that restoration will replace the current dataset.
3. The EXCO member confirms replacement.
4. Laplace restores and saves the snapshot as one operation, then shows the restored records.

Use case ends.

**Extensions**

* 2a. The snapshot is unreadable, incompatible, incomplete, or contains invalid records or broken loan references.
    * 2a1. Laplace rejects it and explains the problem without replacing current data.
    * Use case resumes at step 1.
* 3a. The EXCO member cancels replacement.
    * Use case ends with current data unchanged.
* 4a. The snapshot has changed since validation.
    * 4a1. Laplace discards the previous validation result without replacing current data.
    * Use case resumes at step 2.

#### UC08: Undo a mistaken change

**Scope:** Future. **Related stories:** US37, US41.

**MSS**

1. The EXCO member requests undo of the most recent successful data-changing action.
2. Laplace restores and saves the preceding consistent state, including linked records, and identifies the undone action.
3. The EXCO member inspects the restored records.

Use case ends.

**Extensions**

* 2a. There is no action available to undo.
    * 2a1. Laplace explains that no undo is available and leaves the data unchanged.
    * Use case ends.
The future retention policy must support restoring any deleted records covered by undo.

#### UC09: View a member's current holdings

**Scope:** MVP. **Related stories:** US06, US27, US43.

**MSS**

1. The EXCO member requests a member's details using their NUS ID.
2. Laplace finds the member and shows their details and all equipment linked by open loans, including each item's UUID and expected return date.

Use case ends.

**Extensions**

* 2a. The NUS ID is invalid or no active member has that ID.
    * 2a1. Laplace explains the problem without showing another member's holdings.
    * Use case resumes at step 1.
* 2b. The member has no open loans.
    * 2b1. Laplace shows the member's details and states that they hold no equipment.
    * Use case ends.

#### UC10: View an equipment item's current holder

**Scope:** MVP. **Related stories:** US17, US28, US44.

**MSS**

1. The EXCO member requests an item's details using its UUID.
2. Laplace finds the item and shows its details, current holder's identity and contact details, and the open loan's dates.

Use case ends.

**Extensions**

* 2a. The UUID is invalid or no active item has that UUID.
    * 2a1. Laplace explains the problem without showing another item's details.
    * Use case resumes at step 1.
* 2b. The item has no open loan.
    * 2b1. Laplace shows the item's details and states that it has no current holder.
    * Use case ends.

#### UC11: Export a complete backup

**Scope:** Future. **Related stories:** US03, US41.

**MSS**

1. The EXCO member requests a complete snapshot at a local destination.
2. Laplace creates one snapshot containing the member, equipment, and loan records and their relationships.
3. Laplace reports where the complete snapshot was saved.

Use case ends.

**Extensions**

* 2a. The destination cannot be written or the snapshot cannot be completed.
    * 2a1. Laplace reports the failure and leaves no incomplete snapshot at the destination.
    * Use case resumes at step 1.

#### UC12: Archive an inactive member

**Scope:** Future. **Related stories:** US08, US36, US41.

**MSS**

1. The EXCO member requests archiving of an active member using their NUS ID.
2. Laplace verifies that the member has no open loans.
3. Laplace saves the member as archived, retains their record and closed loans, and removes them from active member lists.

Use case ends.

**Extensions**

* 2a. The NUS ID is invalid or does not identify an active member.
    * 2a1. Laplace explains the problem without changing records.
    * Use case resumes at step 1.
* 2b. The member has one or more open loans.
    * 2b1. Laplace identifies the outstanding equipment and refuses to archive the member.
    * Use case ends. The EXCO member may record the returns in UC04 before trying again.

#### UC13: Recover a deleted record

**Scope:** Future. **Related stories:** US10, US20, US41.
**Precondition:** The future retention policy keeps recoverable deleted records and their linked loans for 30 days.

**MSS**

1. The EXCO member requests recovery of a deleted member or equipment item using its NUS ID or UUID.
2. Laplace finds the deleted record, verifies that it is still within the recovery period, and checks its linked records.
3. Laplace restores and saves the record and its retained loans without creating broken references or an extra open loan.

Use case ends.

**Extensions**

* 2a. The identifier is invalid, the record is not retained, or the 30-day recovery period has elapsed.
    * 2a1. Laplace explains why recovery is unavailable.
    * Use case ends without changing records.
* 2b. Restoring the record would conflict with an active identifier or a linked record cannot be restored consistently.
    * 2b1. Laplace reports the conflict and leaves current records unchanged.
    * Use case ends without recovery.

#### UC14: Redo an undone change

**Scope:** Future. **Related stories:** US38, US41.
**Precondition:** A successful data-changing action has been undone.

**MSS**

1. The EXCO member requests redo of the most recently undone action.
2. Laplace reapplies and saves the action, including its linked records, and identifies the redone action.

Use case ends.

**Extensions**

* 2a. No action is available to redo, including when a new successful data-changing action has invalidated redo.
    * 2a1. Laplace explains that redo is unavailable and leaves current records unchanged.
    * Use case ends.

A new successful data-changing action after undo invalidates redo. Read-only actions and failed commands do not add undo entries or invalidate redo.

### Non-Functional Requirements

These are acceptance targets for the intended product, not measurements of the current implementation. Product constraints are adapted from the [course project constraints](https://nus-cs2103-ay2627-s1.github.io/website/admin/tp-constraints.html); development-process requirements such as incremental delivery are not product NFRs.

1. **NFR01 — Platform compatibility:** The release shall launch and support its documented operations on Windows, Linux, and macOS with Java 25 as the only installed Java version.
2. **NFR02 — Portable distribution:** The application shall run from a single downloadable JAR without an installer or separate installation of dependencies other than Java. The JAR shall not exceed 100 MB.
3. **NFR03 — Local operation:** Member, equipment, loan, and local backup operations shall work without a network connection, remote server, or external account. Each dataset belongs to one local operator; shared or concurrent access is unsupported.
4. **NFR04 — Editable storage:** Persistent records shall use local, human-editable text files without a DBMS. Valid edits made while the application is closed shall load on restart; invalid files shall produce a diagnostic rather than be silently accepted.
5. **NFR05 — Keyboard operation:** Every documented record-management operation shall be completable with keyboard input alone. Users shall not need mouse actions to submit commands, inspect results, or correct input.
6. **NFR06 — Display compatibility:** At 1920×1080 with 100% or 125% scaling, essential controls and feedback shall not overlap or be clipped. At 1280×720 with 150% scaling, every operation and its result shall remain accessible, using scrolling where necessary.
7. **NFR07 — Capacity and response time:** With 1,000 members, 5,000 equipment items, and 10,000 loans on a computer with a four-core CPU, 8 GB RAM, and an SSD, at least 95 of 100 executions of each core add, list, exact lookup, delete, issue, and return operation shall display a result within two seconds of submission. Measure after startup, with valid data and no competing intensive workload; include local saving for mutations. Startup with the same dataset shall complete within ten seconds.
8. **NFR08 — Atomicity and durability:** After a successful mutation and normal shutdown, restarting shall recover the same records and loan relationships. Validation or reported save failures shall not leave partially updated member, equipment, or loan data. A failed file replacement shall preserve the last valid saved dataset. The same guarantee applies to future batch, import, restore, and undo operations as a whole.
9. **NFR09 — Consistent feedback:** Every rejected command shall identify the first detected problem and provide an accepted format or corrective action. Input errors shall not terminate the application, and the submitted command shall remain available for correction. Success and error messages shall be distinguishable by text, without relying on color alone.
10. **NFR10 — Data confidentiality:** Laplace shall not transmit member details or inventory records over the network. Diagnostic logs shall omit member names, phone numbers, email addresses, and raw commands containing those values. Local text data and user-created exports rely on the operating system's file access controls; application-level encryption is not promised.

### Glossary

| Term | Definition |
|------|------------|
| CCA | Co-Curricular Activity; the club or student organization whose records the operator manages. |
| EXCO member / operator | A member of the CCA's executive committee who operates their own Laplace installation. |
| Member record | The stored identity and contact details of a person in the CCA. Being recorded does not give that person application access. |
| NUS ID | A member's unique, immutable identifier in Laplace. The MVP accepts one ASCII letter, seven digits, and an optional final ASCII letter; comparison ignores case and storage uses uppercase. |
| Equipment item / unit | One individually tracked physical object. Identical models are separate items, each with its own identifier. |
| UUID | Universally unique identifier. Laplace generates one immutable UUID for each equipment item; names and descriptions are not equipment identifiers. |
| Category | A broad grouping recorded on an equipment item, such as cameras or sports equipment. One category can contain different equipment types. |
| Equipment type | A particular kind or model of equipment, such as a Canon EOS R50 camera. Future batch addition and per-type summaries require an explicit type field with the same normalized value on units of that type. The MVP records name and category but has no separate type field. |
| Condition | The recorded physical state of an item: `GOOD`, `FAIR`, `DAMAGED`, or `UNDER_REPAIR` in the MVP. |
| Availability | A derived state: `ASSIGNED` when an open loan exists; otherwise `AVAILABLE` for `GOOD` or `FAIR` condition, and `UNAVAILABLE` for `DAMAGED` or `UNDER_REPAIR`. It is not entered independently. |
| Equipment loan / assignment | A relationship between one member and one equipment item, with assignment, expected return, and optional actual return dates. A member may have several loans; an item has at most one open loan. |
| Open loan / outstanding equipment | A loan with no actual return date, or the item covered by that loan. The associated member remains responsible until return is recorded. |
| Closed loan | A loan whose actual return date has been recorded. It no longer determines a current holder. |
| Holder | The member referenced by an item's open loan. An item without an open loan has no current holder. |
| Assignment date | The date the item was issued; it must be a real calendar date no later than the computer's local date. |
| Expected return date | The due date, on or after assignment. Reaching it does not automatically record a return. |
| Actual return date | The recorded date of return, from the assignment date through the computer's local date, inclusive. |
| Active record | A member or equipment record included in normal lists, rather than archived or deleted. Active equipment can still be assigned or unavailable. |
| Archive | A proposed operation that removes an inactive member from the active list while preserving their record and history. |
| Delete | In the MVP, permanent removal of a record and its closed loans, allowed only without open loans. Future 30-day recovery requires retaining deleted data instead. |
| Restore | Either proposed recovery of a deleted record within 30 days, or replacement of the entire dataset from a complete snapshot; the context identifies which operation applies. |
| Snapshot | A complete local export of Laplace's persistent records and their relationships at a point in time, used as a backup. It differs from importing selected member or equipment records. |
| Fuzzy search | Proposed matching of incomplete or approximate text against records; distinct from the MVP's exact identifier lookup. |
| Atomic operation | An operation whose changes succeed together or leave the preceding consistent state intact. |
| MVP | Minimum viable product; the Week 6 baseline, which is a subset of the full requirements recorded here. |

--------------------------------------------------------------------------------------------------------------------

## **Appendix: Instructions for manual testing**

The cases below test the inherited AddressBook Level 3 baseline. Replace them with Laplace member, equipment, and loan cases when those features are implemented; the requirements appendix above specifies their intended behavior.

<box type="info" seamless>

**Note:** These instructions only provide a starting point for testers to work on;
testers are expected to do more *exploratory* testing.
</box>

### Launch and shutdown

1. Initial launch

    1. Download the JAR file and copy it into an empty folder.

    1. Double-click the JAR file.<br>
       Expected: The GUI opens with a set of sample contacts. The window size may not be optimal.

1. Saving window preferences

    1. Resize the window to an optimal size. Move the window to a different location. Close the window.

    1. Relaunch the app by double-clicking the JAR file.<br>
       Expected: The most recent window size and location are retained.

### Deleting a person

1. Deleting a person while all persons are being shown

    1. Prerequisites: List all persons using the `list` command, with multiple persons in the list.

    1. Test case: `delete 1`<br>
       Expected: The first contact is deleted from the list. The status message shows the deleted contact's details.

    1. Test case: `delete 0`<br>
       Expected: No person is deleted. The status message shows error details.

    1. Other incorrect delete commands to try: `delete`, `delete x`, `...` (where x is larger than the list size)<br>
       Expected: Similar to previous.

### Saving data

1. Saving after a successful command

    1. Add a person using the `add` command, then exit and relaunch the app.<br>
       Expected: The added person is still present.
