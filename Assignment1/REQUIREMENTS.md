# CrisisSync – Requirements Specification

**Course:** Engineering Methods for Software Systems Development (61756) – ORT Braude College
**Assignment:** 1 – Operational Analysis, Use Case Model and Requirements Engineering
**Group:** 7
**Repository:** https://github.com/ameerhijazi21-stack/G7-CrisisSync

| Name | ID | College email |
|------|----|---------------|
| Ameer Hijazi | 213910532 | ameer.hijazi@e.braude.ac.il |
| Ayham Awad | 213040876 | Aiham.Awad@e.braude.ac.il |
| Oudai Jumaa | 206977902 | Oudai.Jumaa@e.braude.ac.il |
| Mohamed Diab | 325555217 | Mohamed.diab2@e.braude.ac.il |

---

## 1. Scope

This document specifies the requirements of the CrisisSync emergency-management system.

- Sections 3 and 4 define the functional (FR) and non-functional (NFR) requirements of the **entire system**.
- Section 6 defines the BDD acceptance tests for the **two operations in scope** (staff clarification, Moodle forum, 15.11.2026):
  1. **OP-1:** Reserving a resource package for a task, including reporting task completion.
  2. **OP-2:** Generating the SLA (timeliness and service-level) report.
- Section 5 is the Requirements Traceability Matrix (RTM). It maps every requirement to the use case that realizes it and, for OP-1 and OP-2, to the acceptance test that verifies it.

Business rules clarified by the course staff on the Moodle forum (15.11.2026) are incorporated into the requirements.

## 2. Definitions

| Term | Meaning |
|---|---|
| Resource | A crew member (including a field commander) or an equipment item. A resource is reserved for at most one task at any time. |
| Resource package | The set of resources a task requires. It contains exactly one field commander. |
| Task states | Blocked, Ready, Active, Completed, Cancelled. Completed and Cancelled are final. |
| Event states | Open, In Settlement, Closed, Cancelled. |
| Assignment record | A record of a resource's assignment to a task, with start and end time. Used for cost and utilization calculations. |
| Handling time | The time from event opening until the last Completed task of the event. |
| SLA target | A maximum handling time defined per event type by the System Administrator. |

---

## 3. Functional Requirements

### 3.1 Access and administration

| ID | Requirement | Use case |
|---|---|---|
| FR-01 | The system shall authenticate every user by username and password before granting access to any function. | UC01 Log In |
| FR-02 | The system shall allow each user only the functions permitted for the user's role: System Administrator, Logistics Officer, Dispatcher, Field Commander or Field Crew Member. | UC01 Log In |
| FR-03 | The system shall allow the System Administrator to create, update and deactivate user accounts and to assign a role to each account. | UC02 Manage Users and Permissions |
| FR-04 | The system shall allow the System Administrator to load initial data from an SQL, JSON or CSV file, and shall reject the whole file, listing the invalid records, if any record is invalid. | UC03 Load Initial Data |
| FR-05 | The system shall record every state transition and user action in a read-only audit log, with the timestamp and the user ID. | UC04 View and Filter Event History |
| FR-06 | The system shall allow the System Administrator to filter the audit log by date range, event, event type, user and action type. | UC04 View and Filter Event History |

### 3.2 Resource management

| ID | Requirement | Use case |
|---|---|---|
| FR-07 | The system shall allow the Logistics Officer to set the maintenance status of an equipment item (Available or Faulty) and to transfer an item between stations. | UC08 Manage Equipment and Locations |
| FR-08 | The system shall allow the Logistics Officer to record exceptional unavailability of a crew member (e.g., illness), including a crew member who is currently reserved. | UC09 Update Crew Availability |
| FR-09 | The system shall allow a field crew member or field commander to report shift start and readiness, and shall reject a request to mark themselves unavailable while they are reserved for a task. | UC24 Report Shift Start and Readiness |

### 3.3 Events and tasks

| ID | Requirement | Use case |
|---|---|---|
| FR-10 | The system shall allow the Dispatcher to open an event with a name, an event type (Building Collapse, Fire, Road Accident, Hazmat Leak or Natural Disaster) and a location, and shall record the opening time automatically. | UC10 Create Event |
| FR-11 | The system shall allow the Dispatcher to define a task with a name, an estimated duration, prerequisite tasks and a resource package that contains exactly one field commander. | UC11 Define Tasks and Dependencies |
| FR-12 | The system shall reject any dependency change that creates a cycle, and shall allow dependency changes only for tasks in the Blocked state. | UC11 Define Tasks and Dependencies |
| FR-13 | The system shall move a task from Blocked to Ready when all of its prerequisite tasks are Completed. | UC11 Define Tasks and Dependencies |

### 3.4 Resource reservation and task completion (OP-1)

| ID | Requirement | Use case |
|---|---|---|
| FR-14 | The system shall allow the Dispatcher to reserve a resource package manually only for a task in the Ready state. | UC12 Reserve Resource Package |
| FR-15 | The system shall reserve all resources of a package in a single database transaction with row-level locking ("all or nothing"). | UC12 Reserve Resource Package |
| FR-16 | The system shall reject a reservation that includes a resource whose status is Reserved, Unavailable or Faulty. | UC12 Reserve Resource Package |
| FR-17 | If a reservation transaction fails, the system shall roll it back completely and notify the Dispatcher which resources were unavailable. | UC12 Reserve Resource Package |
| FR-18 | On a successful reservation, the system shall set the task to Active, set every reserved resource to Reserved, open an assignment record for each resource, and push the update to all connected clients. | UC12 Reserve Resource Package |
| FR-19 | The system shall select resources for automatic allocation by a pluggable selection criterion, where the default criterion is the nearest available resource. | UC13 Automatically Allocate Resources |
| FR-20 | If a resource selected by automatic allocation is reserved concurrently, the system shall replace only the taken resources and retry the full package transaction, with at most two alternative attempts (three attempts in total). | UC13 Automatically Allocate Resources |
| FR-21 | If automatic allocation fails, the system shall notify the Dispatcher and state whether no alternative resource existed or all attempts failed due to concurrent reservations. | UC13 Automatically Allocate Resources |
| FR-22 | The system shall allow only the field commander assigned to an Active task to report its completion. | UC22 Report Task Completion |
| FR-23 | The system shall reject a completion report while a package resource is missing, unless the Dispatcher has confirmed that the resource is no longer required. | UC22 Report Task Completion |
| FR-24 | On task completion, the system shall set the task to Completed, record the completion time, release all its resources to Available, close their assignment records, and re-evaluate the dependent tasks. | UC22 Report Task Completion |

### 3.5 Failures and replacements

| ID | Requirement | Use case |
|---|---|---|
| FR-25 | When an equipment failure is reported, the system shall set the item to Faulty, release it from its task, close its assignment record, keep the task Active and alert the Dispatcher. | UC23 Report Equipment Failure |
| FR-26 | When a reserved crew member becomes unavailable, the system shall release the member from the task, close the assignment record, keep the task Active and alert the Dispatcher. | UC09 Update Crew Availability |
| FR-27 | The system shall allow the Dispatcher to assign a replacement resource to an Active task, manually or automatically. If the replaced resource is the field commander, the task permissions shall transfer to the new commander. | UC19 Replace Released Resource |

### 3.6 Cancellation and closing

| ID | Requirement | Use case |
|---|---|---|
| FR-28 | Before cancelling a task, the system shall show the Dispatcher the number of dependent tasks that will also be cancelled. | UC15 Cancel Task |
| FR-29 | On confirmation, the system shall recompute the affected tasks under lock. If the result differs from what was shown, it shall abort and ask for confirmation again. Otherwise it shall cancel the task and its dependents and release their resources. | UC15 Cancel Task |
| FR-30 | The system shall cancel an Open event together with all of its non-final tasks, and shall reject cancelling an event that is In Settlement. | UC16 Cancel Event |
| FR-31 | The system shall allow closing an event only when all of its tasks are in a final state, and shall move the event to In Settlement in a transaction that only one Dispatcher can complete. | UC17 Close Event |
| FR-32 | The system shall calculate the event costs from the assignment records and send one billing request per event, with a unique event ID, to the National Clearing Settlement System. | UC18 Process Settlement |
| FR-33 | On settlement approval, the system shall store the transaction token and close the event. On rejection, failure or timeout, it shall return the event to Open and alert the Dispatcher. | UC18 Process Settlement |

### 3.7 Monitoring

| ID | Requirement | Use case |
|---|---|---|
| FR-34 | The system shall display a live status screen of open events, task trees and task states, updated by server push without polling. Tasks that exceeded their estimated duration shall be highlighted. | UC14 Monitor Open Events |
| FR-35 | The system shall allow field users to view only the events and tasks assigned to them. | UC20 View Assigned Events and Tasks, UC21 View Assigned Task |

### 3.8 Reports (OP-2)

| ID | Requirement | Use case |
|---|---|---|
| FR-36 | The system shall automatically generate the SLA report for the previous calendar month on the first day of every month at 02:00. | UC07 Generate SLA Report |
| FR-37 | The SLA report shall include only events that were finally closed during the report month and that contain at least one Completed task. | UC07 Generate SLA Report |
| FR-38 | The system shall calculate the handling time of each event from its opening time until its last Completed task, ignoring Cancelled tasks. | UC07 Generate SLA Report |
| FR-39 | For each event type and for all types together, the SLA report shall show the number of events, the average handling time and the percentage of events handled within the SLA target. | UC07 Generate SLA Report |
| FR-40 | The system shall allow the System Administrator to generate the SLA report on demand for any previous month. | UC05 View / Generate Monthly Reports |
| FR-41 | If no event qualifies for the report month, the system shall generate the report with zero counts and the message "No qualifying events for this period". | UC07 Generate SLA Report |
| FR-42 | If report generation fails, the system shall retry it up to 3 times at 10-minute intervals, then alert the System Administrator. It shall never store a partial report. | UC07 Generate SLA Report |
| FR-43 | The system shall generate a monthly resource-utilization report with a graphical display. | UC06 Generate Resource Utilization Report |

---

## 4. Non-Functional Requirements

| ID | Category | Requirement |
|---|---|---|
| NFR-01 | Performance | A manual reservation of a package of up to 20 resources shall complete, and its result shall be displayed, within 2 seconds for 95% of requests, with 20 concurrently connected clients. |
| NFR-02 | Performance | A state change shall reach all connected clients by server push within 1 second for 95% of changes. |
| NFR-03 | Performance | The SLA report for a month with up to 5,000 events shall be generated within 10 seconds. |
| NFR-04 | Reliability | In a test of 100 simultaneous reservation attempts for the same resources from at least 2 stations, 0 resources shall be reserved for more than one task. |
| NFR-05 | Reliability | 100% of failed reservation transactions shall leave no partial reservation in the database. |
| NFR-06 | Reliability | A settlement request shall time out after 10 seconds. Repeated requests with the same event ID shall result in at most one charge. |
| NFR-07 | Security | Passwords shall be stored only as salted hashes (bcrypt or PBKDF2). After 5 consecutive failed log-in attempts, the account shall be locked for 5 minutes. |
| NFR-08 | Security | The server shall verify role permissions for 100% of requests. Unauthorized requests shall be rejected and recorded in the audit log. |
| NFR-09 | Security | No application function shall allow modifying or deleting an audit-log entry. |
| NFR-10 | Usability | A trained Dispatcher shall be able to complete a manual reservation within 60 seconds and in at most 5 user actions after selecting the task. |
| NFR-11 | Usability | Every error message shall state the cause and the affected item by name (e.g., the unavailable resource). No message shall show a technical error code alone. |
| NFR-12 | Maintainability | Adding a new allocation criterion or a new report type shall require implementing one new class against an existing interface, with no change to existing classes. |
| NFR-13 | Portability | The client shall be a Java/JavaFX desktop application that communicates with the server through OCSF over TCP/IP on the local network. |

---

## 5. Requirements Traceability Matrix (RTM)

Acceptance tests cover OP-1 and OP-2, as instructed by the course staff. The remaining requirements are traced to their use cases.

| Requirement | Use case | Acceptance test(s) |
|---|---|---|
| FR-14 | UC12 Reserve Resource Package | AT-01, AT-06 |
| FR-15 | UC12 Reserve Resource Package | AT-01, AT-03 |
| FR-16 | UC12 Reserve Resource Package | AT-02 |
| FR-17 | UC12 Reserve Resource Package | AT-02, AT-10 |
| FR-18 | UC12 Reserve Resource Package | AT-01 |
| FR-19 | UC13 Automatically Allocate Resources | AT-04 |
| FR-20 | UC13 Automatically Allocate Resources | AT-04, AT-05 |
| FR-21 | UC13 Automatically Allocate Resources | AT-05 |
| FR-22 | UC22 Report Task Completion | AT-09 |
| FR-23 | UC22 Report Task Completion | AT-08 |
| FR-24 | UC22 Report Task Completion | AT-07 |
| FR-36 | UC07 Generate SLA Report | AT-11 |
| FR-37 | UC07 Generate SLA Report | AT-11, AT-12 |
| FR-38 | UC07 Generate SLA Report | AT-11, AT-12 |
| FR-39 | UC07 Generate SLA Report | AT-11 |
| FR-40 | UC05 View / Generate Monthly Reports | AT-13 |
| FR-41 | UC07 Generate SLA Report | AT-14 |
| FR-42 | UC07 Generate SLA Report | AT-15 |
| NFR-01 | UC12 Reserve Resource Package | AT-01 |
| NFR-03 | UC07 Generate SLA Report | AT-16 |
| NFR-04 | UC12 Reserve Resource Package | AT-03 |
| NFR-05 | UC12 Reserve Resource Package | AT-02, AT-10 |
| NFR-08 | UC22 Report Task Completion, UC05 View / Generate Monthly Reports | AT-09, AT-13 |
| NFR-11 | UC12 Reserve Resource Package | AT-02 |
| FR-01 – FR-02 | UC01 Log In | Outside acceptance scope |
| FR-03 | UC02 Manage Users and Permissions | Outside acceptance scope |
| FR-04 | UC03 Load Initial Data | Outside acceptance scope |
| FR-05 – FR-06 | UC04 View and Filter Event History | Outside acceptance scope |
| FR-07 | UC08 Manage Equipment and Locations | Outside acceptance scope |
| FR-08, FR-26 | UC09 Update Crew Availability | Outside acceptance scope |
| FR-09 | UC24 Report Shift Start and Readiness | Outside acceptance scope |
| FR-10 | UC10 Create Event | Outside acceptance scope |
| FR-11 – FR-13 | UC11 Define Tasks and Dependencies | Outside acceptance scope |
| FR-25 | UC23 Report Equipment Failure | Outside acceptance scope |
| FR-27 | UC19 Replace Released Resource | Outside acceptance scope |
| FR-28 – FR-29 | UC15 Cancel Task | Outside acceptance scope |
| FR-30 | UC16 Cancel Event | Outside acceptance scope |
| FR-31 | UC17 Close Event | Outside acceptance scope |
| FR-32 – FR-33 | UC18 Process Settlement | Outside acceptance scope |
| FR-34 | UC14 Monitor Open Events | Outside acceptance scope |
| FR-35 | UC20, UC21 | Outside acceptance scope |
| FR-43 | UC06 Generate Resource Utilization Report | Outside acceptance scope |
| NFR-02, NFR-06, NFR-07, NFR-09, NFR-10, NFR-12, NFR-13 | System-wide | Outside acceptance scope |

---

## 6. Acceptance Tests (BDD)

### OP-1 – Reserve a resource package and report task completion

```gherkin
Feature: Resource package reservation and task completion

  Background:
    Given the Dispatcher "dana" is logged in
    And the event "Fire – Karmiel Industrial Zone" is Open
    And the task "Evacuate building A" belongs to the event

  # AT-01 – Normal flow
  Scenario: AT-01 Successful manual reservation of a full package
    Given the task "Evacuate building A" is in state Ready
    And the field commander "Commander Levi", the paramedic "Medic Cohen" and the ambulance "AMB-12" are Available
    When the Dispatcher reserves the package ["Commander Levi", "Medic Cohen", "AMB-12"] for the task
    Then all three resources have status Reserved
    And the task state is Active
    And an assignment record with a start time is opened for each resource
    And all connected clients show the task as Active within 1 second
    And the Dispatcher sees the result within 2 seconds

  # AT-02 – Exception: one resource unavailable, full rollback
  Scenario: AT-02 Reservation is rolled back when one resource is Faulty
    Given the task "Evacuate building A" is in state Ready
    And the ambulance "AMB-12" has status Faulty
    When the Dispatcher reserves the package ["Commander Levi", "Medic Cohen", "AMB-12"] for the task
    Then no resource of the package is Reserved
    And the task state remains Ready
    And the Dispatcher sees the message "Reservation failed: AMB-12 is Faulty"

  # AT-03 – Concurrency
  Scenario: AT-03 Two stations reserve the same resource at the same time
    Given the tasks "Evacuate building A" and "Search building B" are both Ready
    And "Commander Levi" is Available
    When two Dispatchers at two stations reserve packages that include "Commander Levi" at the same moment
    Then exactly one reservation succeeds
    And "Commander Levi" is reserved for exactly one task
    And the other Dispatcher is notified that "Commander Levi" is no longer available
    And the task of the failed reservation remains Ready

  # AT-04 – Automatic allocation with an alternative attempt
  Scenario: AT-04 Automatic allocation recovers by replacing a taken resource
    Given the task "Evacuate building A" is Ready
    And the nearest ambulance "AMB-12" is reserved by another station during the allocation
    And the ambulance "AMB-15" is Available
    When the Dispatcher requests automatic allocation for the task
    Then the system replaces only "AMB-12" with "AMB-15" and keeps the other selected resources
    And the reservation succeeds within 3 attempts
    And the task state is Active

  # AT-05 – Automatic allocation fails after all attempts
  Scenario: AT-05 Automatic allocation fails after three attempts due to contention
    Given the task "Evacuate building A" is Ready
    And on every attempt one selected resource is reserved concurrently by another station
    When the Dispatcher requests automatic allocation for the task
    Then the system makes exactly 3 reservation attempts
    And no resource is reserved for the task
    And the Dispatcher sees the message "Allocation failed: resources were taken by concurrent reservations"

  # AT-06 – Exception: task not Ready
  Scenario: AT-06 Reservation is rejected for a Blocked task
    Given the task "Evacuate building A" is in state Blocked
    When the Dispatcher tries to reserve a package for the task
    Then the reservation is rejected
    And the Dispatcher sees the message "Task is not Ready: prerequisite tasks are not completed"

  # AT-07 – Normal flow: completion
  Scenario: AT-07 Field commander reports task completion
    Given the task "Evacuate building A" is Active with "Commander Levi" as field commander
    And the task "Search building B" depends only on "Evacuate building A" and is Blocked
    When "Commander Levi" reports completion of "Evacuate building A"
    Then the task state is Completed and the completion time is recorded
    And all resources of the package are Available
    And their assignment records are closed with an end time
    And the task "Search building B" becomes Ready

  # AT-08 – Exception: missing resource
  Scenario: AT-08 Completion is rejected while a package resource is missing
    Given the task "Evacuate building A" is Active
    And the ambulance "AMB-12" was released from the task after a failure report
    And no replacement was assigned and the Dispatcher did not confirm it is not required
    When "Commander Levi" reports completion of the task
    Then the report is rejected
    And the task remains Active
    And "Commander Levi" sees the message "Cannot complete: resource AMB-12 is missing"

  # AT-09 – Security
  Scenario: AT-09 A commander who is not assigned cannot report completion
    Given the task "Evacuate building A" is Active with "Commander Levi" as field commander
    When the field commander "Commander Mizrahi" reports completion of the task
    Then the report is rejected as unauthorized
    And the task remains Active
    And the rejected attempt is recorded in the audit log

  # AT-10 – Recovery from a failure during the transaction
  Scenario: AT-10 Database failure during reservation is fully rolled back
    Given the task "Evacuate building A" is Ready
    And the database connection fails after the first resource is locked
    When the Dispatcher reserves the package for the task
    Then no resource of the package is Reserved
    And the task state remains Ready
    And the Dispatcher sees the message "Reservation failed: please try again"
    And a new reservation attempt after the connection is restored succeeds
```

### OP-2 – Generate the SLA report

```gherkin
Feature: SLA report generation

  Background:
    Given the SLA target for event type "Fire" is 4 hours

  # AT-11 – Normal flow: automatic monthly report
  Scenario: AT-11 Monthly SLA report is generated automatically
    Given in October 2026 two "Fire" events were finally closed
    And event E1 was opened at 08:00 and its last Completed task finished at 11:00
    And event E2 was opened at 09:00 and its last Completed task finished at 15:00
    When the Monthly Scheduler triggers report generation on 1 November 2026 at 02:00
    Then the SLA report for October 2026 is stored
    And for type "Fire" it shows 2 events, an average handling time of 4.5 hours and 50% within the SLA target

  # AT-12 – Filtering rules
  Scenario: AT-12 Cancelled events and Cancelled tasks are excluded
    Given in October 2026 the event E3 was cancelled
    And the closed event E4 was opened at 08:00, its last Completed task finished at 10:00, and a Cancelled task ended at 13:00
    And the event E5 was closed in October 2026 with only Cancelled tasks
    When the SLA report for October 2026 is generated
    Then E3 and E5 are not included in the report
    And the handling time of E4 is 2 hours

  # AT-13 – On-demand generation and authorization
  Scenario: AT-13 System Administrator generates a report on demand for a previous month
    Given the System Administrator is logged in
    When the System Administrator requests the SLA report for September 2026
    Then the SLA report for September 2026 is displayed
    But when a Dispatcher requests the same report
    Then the request is rejected as unauthorized and recorded in the audit log

  # AT-14 – Edge case: empty period
  Scenario: AT-14 Report for a month without qualifying events
    Given no event was finally closed in August 2026
    When the SLA report for August 2026 is generated
    Then the report shows 0 events for every type
    And it displays the message "No qualifying events for this period"

  # AT-15 – Recovery after a generation failure
  Scenario: AT-15 Generation failure is retried and then reported
    Given the database is unavailable on 1 November 2026 at 02:00
    When the Monthly Scheduler triggers the SLA report for October 2026
    Then no partial report is stored
    And the system retries the generation every 10 minutes, up to 3 times
    And if all retries fail, the System Administrator receives an alert "SLA report for October 2026 failed"
    And if the database is restored before the last retry, the full report is stored

  # AT-16 – Performance
  Scenario: AT-16 Report for a large month is generated in time
    Given 5,000 events were finally closed in October 2026
    When the SLA report for October 2026 is generated
    Then the report is stored within 10 seconds
```

---

## 7. Assumptions

1. The SLA target per event type is configured by the System Administrator, because the system story does not define target values.
2. An event belongs to the report month in which it was finally closed (staff clarification).
3. The National Clearing Settlement System is simulated by a mock adapter that supports rejection, timeout and duplicate-request detection.
