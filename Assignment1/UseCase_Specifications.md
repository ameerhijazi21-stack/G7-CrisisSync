# CrisisSync – Use Case Specifications (Group 7)

Paste each section into Visual Paradigm:
- **Preconditions / Post-conditions** → Use Case Details → *Details* tab.
- **Main Flow** → *Flow of Events* tab, one step per line.
- **Extensions** → *Extension* pane, attached to the step number shown.

Full specifications are given for the main use cases. Included and extending use cases (UC06, UC18, UC26, UC25) get a short description only, as clarified on the course forum.

---

## UC01 – Log In
**Actors:** All human actors
**Preconditions:** The user has an active account. The client is connected to the server.
**Post-conditions (success):** The user is authenticated and sees the main screen of their role.
**Post-conditions (failure):** The user is not authenticated. The failed attempt is recorded in the audit log.

**Main Flow**
1. The user opens the CrisisSync client.
2. The system displays the log-in screen.
3. The user enters a username and password and submits.
4. The system validates the credentials.
5. The system identifies the user's role.
6. The system opens the main screen of that role and records the log-in in the audit log.

**Extensions**
- 4a. The credentials are invalid: the system shows "Invalid username or password" and returns to step 3.
- 4b. Five consecutive attempts failed: the system locks the account for 5 minutes and informs the user.
- 4c. The account is deactivated: the system shows "Account is inactive – contact the administrator".

---

## UC02 – Manage Users and Permissions
**Actor:** System Administrator
**Preconditions:** The System Administrator is logged in.
**Post-conditions (success):** The account is created, updated or deactivated with the selected role, and the change is recorded in the audit log.
**Post-conditions (failure):** No account data is changed.

**Main Flow**
1. The System Administrator opens user management.
2. The system displays the list of user accounts.
3. The System Administrator chooses to create, edit or deactivate an account.
4. The System Administrator enters or updates the account details and selects a role.
5. The System Administrator saves.
6. The system validates the data, saves the account and records the action.

**Extensions**
- 6a. The username already exists: the system shows an error and returns to step 4.
- 6b. A required field is missing: the system highlights the field and returns to step 4.
- 3a. The System Administrator tries to deactivate their own account: the system rejects the action.

---

## UC03 – Load Initial Data
**Actor:** System Administrator
**Preconditions:** The System Administrator is logged in. A data file in SQL, JSON or CSV format is available.
**Post-conditions (success):** All records in the file are loaded into the database.
**Post-conditions (failure):** No record is loaded.

**Main Flow**
1. The System Administrator opens the data-loading utility.
2. The System Administrator selects a file and its format.
3. The system validates all records in the file.
4. The system loads all records in one transaction.
5. The system shows the number of loaded records and records the action in the audit log.

**Extensions**
- 3a. One or more records are invalid: the system rejects the whole file and lists the invalid records with their line numbers.
- 3b. The file format is not supported: the system shows an error and returns to step 2.

---

## UC04 – View and Filter Event History
**Actor:** System Administrator
**Preconditions:** The System Administrator is logged in.
**Post-conditions (success):** The matching audit-log entries are displayed. The log is not changed.

**Main Flow**
1. The System Administrator opens the event history.
2. The system displays the latest audit-log entries.
3. The System Administrator sets filters: date range, event, event type, user or action type.
4. The system displays the matching entries with timestamp, user and action.
5. The System Administrator selects an entry to view its details.

**Extensions**
- 4a. No entry matches the filters: the system shows "No matching entries".

---

## UC05 – View / Generate Monthly Reports
**Actor:** System Administrator
**Includes:** UC06 Generate Resource Utilization Report, UC07 Generate SLA Report
**Preconditions:** The System Administrator is logged in.
**Post-conditions (success):** The requested report is displayed. A newly generated report is stored.
**Post-conditions (failure):** No partial report is stored.

**Main Flow**
1. The System Administrator opens the reports screen.
2. The system displays the list of stored monthly reports.
3. The System Administrator selects a report type and a previous month.
4. If the report is not stored, the system generates it (include UC06 or UC07).
5. The system displays the report.

**Extensions**
- 3a. The selected month is the current or a future month: the system rejects the request.
- 4a. Generation fails: the system shows an error and stores nothing.

---

## UC06 – Generate Resource Utilization Report *(included)*
Generates the monthly resource-utilization report with a graphical display, from the assignment records of the month. It is triggered by the Monthly Scheduler or included by UC05.

---

## UC07 – Generate SLA Report
**Actor:** Monthly Scheduler (primary). It is also included by UC05.
**Preconditions:** The 1st day of the month at 02:00 has arrived, or UC05 requested the report. SLA targets are defined for each event type.
**Post-conditions (success):** The SLA report for the month is stored and available to the System Administrator.
**Post-conditions (failure):** No partial report is stored, and the System Administrator is alerted.

**Main Flow**
1. The Monthly Scheduler triggers SLA report generation for the previous month.
2. The system selects the events that were finally closed in that month and have at least one Completed task.
3. For each event, the system calculates the handling time from event opening to the last Completed task, ignoring Cancelled tasks.
4. For each event type and in total, the system calculates the number of events, the average handling time and the percentage within the SLA target.
5. The system stores the report and records the action in the audit log.

**Extensions**
- 2a. No event qualifies: the system stores a report with zero counts and the message "No qualifying events for this period".
- 2b. The database is unavailable: the system retries every 10 minutes, up to 3 times. If all retries fail, it alerts the System Administrator.

---

## UC08 – Manage Equipment and Locations
**Actor:** Logistics Officer
**Preconditions:** The Logistics Officer is logged in.
**Post-conditions (success):** The equipment status or station is updated, the change is pushed to the clients and recorded in the audit log.

**Main Flow**
1. The Logistics Officer opens the equipment list.
2. The system displays the equipment with type, station and status.
3. The Logistics Officer selects an item.
4. The Logistics Officer changes its maintenance status (Available / Faulty) or transfers it to another station.
5. The system saves the change and pushes the update.

**Extensions**
- 4a. The item is Reserved and is set to Faulty: the system releases it from its task, closes the assignment record, keeps the task Active and alerts the Dispatcher.
- 4b. The item is Reserved and a transfer is requested: the system rejects the transfer.

---

## UC09 – Update Crew Availability
**Actor:** Logistics Officer
**Preconditions:** The Logistics Officer is logged in.
**Post-conditions (success):** The crew member's availability is updated and recorded.

**Main Flow**
1. The Logistics Officer opens the crew list.
2. The Logistics Officer selects a crew member.
3. The Logistics Officer sets an exceptional availability status (e.g., unavailable due to illness) with a reason.
4. The system saves the status, pushes the update and records it in the audit log.

**Extensions**
- 4a. The crew member is reserved for an Active task: the system releases the member, closes the assignment record, keeps the task Active and alerts the Dispatcher to assign a replacement.
- 4b. The crew member is the task's field commander: the alert states that a replacement field commander is mandatory.

---

## UC10 – Create Event
**Actor:** Dispatcher
**Preconditions:** The Dispatcher is logged in.
**Post-conditions (success):** A new Open event exists, with the opening time recorded automatically.

**Main Flow**
1. The Dispatcher chooses to open a new event.
2. The system displays the event form.
3. The Dispatcher enters a name, selects an event type and enters a location.
4. The Dispatcher submits.
5. The system creates the event, records the opening time and pushes it to the live status screen.

**Extensions**
- 4a. A required field is missing: the system highlights it and returns to step 3.

---

## UC11 – Define Tasks and Dependencies
**Actor:** Dispatcher
**Preconditions:** The Dispatcher is logged in. The event is Open.
**Post-conditions (success):** The task tree is saved as a DAG, and every task's state is set correctly (Blocked or Ready).
**Post-conditions (failure):** The task tree is not changed.

**Main Flow**
1. The Dispatcher opens the event's task tree.
2. The Dispatcher adds a task with a name, an estimated duration and a required resource package with exactly one field commander.
3. The Dispatcher selects the prerequisite tasks.
4. The system locks the event and validates that the dependencies form no cycle.
5. The system saves the task, sets it to Blocked or Ready, and pushes the update.

**Extensions**
- 4a. The change creates a cycle: the system rejects it and shows the tasks that form the cycle.
- 3a. The Dispatcher changes the dependencies of a task that is not Blocked: the system rejects the change.
- 2a. The package has no field commander, or more than one: the system rejects it.

---

## UC12 – Reserve Resource Package
**Actor:** Dispatcher (also included by UC13)
**Preconditions:** The Dispatcher is logged in. The task is Ready.
**Post-conditions (success):** All package resources are Reserved, the task is Active, assignment records are open, and all clients are updated.
**Post-conditions (failure):** No resource is reserved, and the task stays Ready.

**Main Flow**
1. The Dispatcher selects a Ready task.
2. The system displays the required package and the available resources.
3. The Dispatcher selects a resource for each package slot.
4. The Dispatcher confirms the reservation.
5. The system locks the event, the task and the selected resources in a fixed order.
6. The system verifies that every resource is Available.
7. The system reserves all resources in one transaction, sets the task to Active and opens the assignment records.
8. The system pushes the update to all clients and confirms to the Dispatcher.

**Extensions**
- 6a. A resource is Reserved, Unavailable or Faulty: the system rolls back, names the resource and returns to step 3.
- 7a. The transaction fails (database error or lock timeout): the system rolls back completely and asks the Dispatcher to try again.
- 1a. The task is not Ready: the system rejects the reservation.

---

## UC13 – Automatically Allocate Resources
**Actor:** Dispatcher
**Includes:** UC12 Reserve Resource Package
**Extended by:** UC26 Retry Allocation with Alternative
**Preconditions:** The Dispatcher is logged in. The task is Ready.
**Post-conditions (success):** The package is reserved as in UC12.
**Post-conditions (failure):** No resource is reserved, and the Dispatcher is told the reason.

**Main Flow**
1. The Dispatcher selects a Ready task and requests automatic allocation.
2. The system applies the active selection criterion (default: nearest available resource) to choose a resource for each slot.
3. The system reserves the package (include UC12).
4. The system shows the allocated resources to the Dispatcher.

**Extensions**
- 2a. A slot has no available resource: the system fails and reports "No alternative resource available".
- 3a. A chosen resource was taken concurrently: extension point "Retry Allocation with Alternative" (extend UC26).

---

## UC14 – Monitor Open Events
**Actor:** Dispatcher
**Preconditions:** The Dispatcher is logged in.
**Post-conditions (success):** The Dispatcher sees the current state of all open events.

**Main Flow**
1. The Dispatcher opens the live status screen.
2. The system displays all open events with their progress, task trees and task states.
3. The system pushes every state change to the screen without polling.
4. The system highlights tasks that exceeded their estimated duration.
5. The Dispatcher selects an event to view its details.

**Extensions**
- 3a. The connection to the server is lost: the system shows "Disconnected – reconnecting" and reloads the full state after reconnecting.

---

## UC15 – Cancel Task
**Actor:** Dispatcher
**Preconditions:** The Dispatcher is logged in. The task is not in a final state, and the event is Open.
**Post-conditions (success):** The task and all its dependent tasks are Cancelled, and their resources are released.
**Post-conditions (failure):** No task is changed.

**Main Flow**
1. The Dispatcher selects a task and chooses to cancel it.
2. The system shows the number of dependent tasks that will also be cancelled (preview).
3. The Dispatcher confirms.
4. The system locks the event and recalculates the affected tasks.
5. The system cancels the task and its dependents, releases their resources, closes the assignment records and pushes the update.

**Extensions**
- 4a. The recalculated number differs from the preview: the system aborts and returns to step 2 with the updated number.
- 3a. The Dispatcher does not confirm: nothing is changed.

---

## UC16 – Cancel Event
**Actor:** Dispatcher
**Preconditions:** The Dispatcher is logged in. The event is Open.
**Post-conditions (success):** The event and all its non-final tasks are Cancelled, and all their resources are released.

**Main Flow**
1. The Dispatcher selects an event and chooses to cancel it.
2. The system shows the number of non-final tasks that will be cancelled.
3. The Dispatcher confirms.
4. The system locks the event, cancels it and its non-final tasks, releases the resources and pushes the update.

**Extensions**
- 4a. The event is In Settlement: the system rejects the cancellation.

---

## UC17 – Close Event
**Actor:** Dispatcher
**Includes:** UC18 Process Settlement
**Extended by:** UC25 Handle Settlement Rejection
**Preconditions:** The Dispatcher is logged in. The event is Open.
**Post-conditions (success):** The event is Closed, and the settlement transaction token is stored.
**Post-conditions (failure):** The event is Open again, and the Dispatcher is alerted.

**Main Flow**
1. The Dispatcher selects an event and chooses to close it.
2. The system locks the event and verifies that all its tasks are in a final state.
3. The system sets the event to In Settlement.
4. The system calculates the event costs from the assignment records.
5. The system sends the settlement request (include UC18).
6. The system receives approval, stores the transaction token and sets the event to Closed.
7. The system pushes the update and records the action in the audit log.

**Extensions**
- 2a. A task is not in a final state: the system rejects the request and lists the open tasks.
- 3a. Another Dispatcher already moved the event to In Settlement: the system informs the Dispatcher and stops.
- 6a. The settlement is rejected, fails or times out: extension point "Handle Settlement Rejection" (extend UC25).

---

## UC18 – Process Settlement *(included)*
The system sends a billing request with the unique event ID and the calculated costs to the National Clearing Settlement System, and returns its answer (approved with a token, rejected, or timeout). Repeated requests with the same event ID are not charged twice.

---

## UC19 – Replace Released Resource
**Actor:** Dispatcher
**Preconditions:** The Dispatcher is logged in. A resource was released from an Active task (equipment failure or crew unavailability).
**Post-conditions (success):** A replacement resource is Reserved for the task, with a new assignment record.

**Main Flow**
1. The system alerts the Dispatcher that a task is missing a resource.
2. The Dispatcher opens the task.
3. The Dispatcher selects a replacement resource, or requests automatic selection.
4. The system reserves the replacement in a transaction and opens an assignment record.
5. The system pushes the update.

**Extensions**
- 3a. The released resource is no longer needed: the Dispatcher confirms this, and the task can be completed without it.
- 4a. The replacement is taken concurrently: the system rolls back and returns to step 3.
- 4b. The replaced resource was the field commander: the system transfers the task permissions to the new commander.

---

## UC20 – View Assigned Events and Tasks
**Actor:** Field Commander
**Preconditions:** The Field Commander is logged in.
**Post-conditions (success):** The Field Commander sees only the events and tasks assigned to them.

**Main Flow**
1. The Field Commander opens "My Tasks".
2. The system displays the events and tasks assigned to the commander, with states and resources.
3. The system pushes updates as they occur.

**Extensions**
- 2a. Nothing is assigned: the system shows "No assigned tasks".

---

## UC21 – View Assigned Task
**Actor:** Field Crew Member
**Preconditions:** The Field Crew Member is logged in.
**Post-conditions (success):** The Field Crew Member sees the task they are assigned to.

**Main Flow**
1. The Field Crew Member opens "My Task".
2. The system displays the assigned task with its event, location, commander and state.

**Extensions**
- 2a. No task is assigned: the system shows "No assigned task".

---

## UC22 – Report Task Completion
**Actor:** Field Commander
**Preconditions:** The Field Commander is logged in and is the commander of the Active task.
**Post-conditions (success):** The task is Completed, its resources are Available, the assignment records are closed and the dependent tasks are re-evaluated.
**Post-conditions (failure):** The task remains Active.

**Main Flow**
1. The Field Commander selects the Active task.
2. The Field Commander reports completion.
3. The system locks the event and verifies that the commander is assigned to the task and that no package resource is missing.
4. The system sets the task to Completed and records the completion time.
5. The system releases all resources and closes the assignment records.
6. The system moves dependent tasks whose prerequisites are all Completed to Ready.
7. The system pushes the update to all clients.

**Extensions**
- 3a. A package resource is missing and the Dispatcher did not confirm it is not needed: the system rejects the report and names the missing resource.
- 3b. The user is not the assigned commander: the system rejects the report and records it in the audit log.

---

## UC23 – Report Equipment Failure
**Actors:** Field Commander, Field Crew Member
**Preconditions:** The user is logged in and assigned to an Active task that uses the equipment item.
**Post-conditions (success):** The item is Faulty and released from the task, the task stays Active, and the Dispatcher is alerted.

**Main Flow**
1. The user selects the equipment item of the task.
2. The user reports a failure with a short description.
3. The system locates the task the item is reserved for, then locks the event, the task and the item.
4. The system sets the item to Faulty, releases it and closes its assignment record.
5. The system keeps the task Active and alerts the Dispatcher to assign a replacement.

**Extensions**
- 3a. The item is no longer reserved for that task: the system informs the user and stops.

---

## UC24 – Report Shift Start and Readiness
**Actors:** Field Crew Member, Field Commander
**Preconditions:** The user is logged in.
**Post-conditions (success):** The user's availability status is updated.

**Main Flow**
1. The user opens the shift screen.
2. The user reports shift start and readiness, or the end of readiness.
3. The system updates the user's status to Available (or Unavailable) and pushes the change.

**Extensions**
- 3a. The user is reserved for a task and reports Unavailable: the system rejects it and tells the user to contact the Logistics Officer.

---

## UC26 – Retry Allocation with Alternative *(extends UC13 at "Retry Allocation with Alternative")*
The system replaces only the resources that were taken and retries the full-package transaction, with at most two alternative attempts (three in total). If all attempts fail, it reports "Allocation failed: resources were taken by concurrent reservations".

## UC25 – Handle Settlement Rejection *(extends UC17 at "Handle Settlement Rejection")*
If the settlement is rejected, fails or times out, the system returns the event to Open and alerts the Dispatcher with the reason.
