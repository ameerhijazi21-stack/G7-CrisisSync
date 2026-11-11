# CrisisSync — Requirements Specification (Assignment 1, Part B)

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

This document specifies the requirements for two CrisisSync operations:

1. **Reserving a resource package for a task**, including the Field Commander's report of task completion.
2. **Generating the SLA (time-to-resolution) report.**

**Assumption A1:** The SLA report for a month includes every event whose last task reached a final state during that month. Cancelled events are excluded.

---

## 2. Functional Requirements

### 2.1 Resource package reservation and task completion

| ID | Requirement |
|----|-------------|
| FR-01 | The system shall allow a Dispatcher to reserve a resource package only for a task in the **Ready** state. |
| FR-02 | The system shall reserve all resources of the package in a single SQL transaction that locks the requested resource records, verifies that each one is Available and sets each one to Reserved. |
| FR-03 | If any resource in the package is not Available, the system shall roll back the entire transaction, keep the task in the Ready state and show the Dispatcher which resource is unavailable. |
| FR-04 | Upon a successful reservation, the system shall change the task state to **Active**. |
| FR-05 | When two Dispatchers request the same resource at the same time, the system shall grant it to exactly one of them, notify the other one and refresh the other one's screen to the current state. |
| FR-06 | The system shall allow only the Field Commander assigned to a task to report its completion. |
| FR-07 | Upon a completion report, the system shall change the task state to **Completed**, set all of the task's resources to Available, and move to Ready every dependent task whose prerequisite tasks are all Completed. |
| FR-08 | The system shall record every reservation, task state change and resource release in the audit log, with its timestamp and the acting user. |

### 2.2 SLA report

| ID | Requirement |
|----|-------------|
| FR-09 | The system shall automatically generate the SLA report at the end of each calendar month, according to the system clock. |
| FR-10 | The SLA report shall show, for each event type, the average time from the opening of an event until the completion of its last task. |
| FR-11 | The system shall allow the System Administrator to generate the SLA report on demand for a selected past month. |
| FR-12 | The system shall allow only the System Administrator to view SLA reports. |

---

## 3. Non-Functional Requirements

| ID | Category | Requirement |
|----|----------|-------------|
| NFR-01 | Performance | The system shall display the result of a reservation (success or failure) within **2 seconds** for at least **95%** of requests, with **20** workstations connected. |
| NFR-02 | Performance | The server shall push every task or resource status change to all relevant connected clients within **1 second**, without client polling. |
| NFR-03 | Performance | The system shall generate an SLA report for a month with up to **1,000** events within **10 seconds**. |
| NFR-04 | Reliability | Under **100** simultaneous reservation attempts for the same resource, the system shall produce **0** double reservations. |
| NFR-05 | Reliability | If the server or database connection fails during a reservation, **100%** of the involved resources shall keep their previous status after recovery. |
| NFR-06 | Security | The system shall reject **100%** of requests from users whose role lacks permission for the operation, and shall store passwords only as salted hashes. |
| NFR-07 | Usability | A trained Dispatcher shall complete a manual reservation from the event status screen in at most **4 UI actions** and within **30 seconds**. |

---

## 4. Requirements Traceability Matrix (RTM)

| Requirement | Use Case | Acceptance Test |
|-------------|----------|-----------------|
| FR-01 | Reserve Resource Package | AT-01 |
| FR-02 | Reserve Resource Package | AT-01 |
| FR-03 | Reserve Resource Package | AT-02 |
| FR-04 | Reserve Resource Package | AT-01 |
| FR-05 | Reserve Resource Package | AT-03 |
| FR-06 | Report Task Completion | AT-05 |
| FR-07 | Report Task Completion | AT-04 |
| FR-08 | Reserve Resource Package, Report Task Completion | AT-01, AT-04 |
| FR-09 | Generate Monthly Reports | AT-06 |
| FR-10 | Generate SLA Report | AT-06 |
| FR-11 | Generate SLA Report | AT-07 |
| FR-12 | Generate SLA Report | AT-12 |
| NFR-01 | Reserve Resource Package | AT-08 |
| NFR-02 | Reserve Resource Package, Report Task Completion | AT-09 |
| NFR-03 | Generate SLA Report | AT-10 |
| NFR-04 | Reserve Resource Package | AT-03 |
| NFR-05 | Reserve Resource Package | AT-11 |
| NFR-06 | Login | AT-12 |
| NFR-07 | Reserve Resource Package | AT-13 |

---

## 5. Acceptance Tests (BDD)

### 5.1 Resource package reservation

#### AT-01 — Successful reservation (normal flow)
```gherkin
# AWAITING: Oudai Jumaa
Given ...
When ...
Then ...
```

#### AT-02 — One resource unavailable, full rollback (error and recovery flow)
```gherkin
# AWAITING: Oudai Jumaa
Given ...
When ...
Then ...
```

#### AT-03 — Two Dispatchers reserve the same resource at the same time
```gherkin
# AWAITING: Oudai Jumaa
Given ...
When ...
Then ...
```

### 5.2 Task completion

#### AT-04 — Assigned Field Commander reports completion (normal flow)
```gherkin
# AWAITING: Mohamed Diab
Given ...
When ...
Then ...
```

#### AT-05 — Another Field Commander tries to complete the task (error flow)
```gherkin
# AWAITING: Mohamed Diab
Given ...
When ...
Then ...
```

### 5.3 SLA report

#### AT-06 — Automatic end-of-month report (normal flow)
```gherkin
# AWAITING: Ayham Awad
Given ...
When ...
Then ...
```

#### AT-07 — Database timeout during on-demand report (error and recovery flow)
```gherkin
# AWAITING: Ayham Awad
Given ...
When ...
Then ...
```

### 5.4 Non-functional tests

#### AT-08 — Reservation response time
```gherkin
Given 20 Dispatcher workstations are connected to the server
When the Dispatchers send 100 reservation requests within 60 seconds
Then at least 95 of them shall return a result within 2 seconds
```

#### AT-09 — Server push
```gherkin
Given a Dispatcher and Field Commander "FC-1" are viewing the status screen of the same event
When "FC-1" reports the completion of a task
Then both screens shall show the task as Completed within 1 second
  And neither client shall have sent a polling request
```

#### AT-10 — SLA report generation time
```gherkin
Given the database contains 1,000 closed events in October 2026
When the System Administrator requests the SLA report for October 2026
Then the report shall be displayed within 10 seconds
```

#### AT-11 — Server failure during reservation (recovery flow)
```gherkin
Given task "Debris Removal" is Ready and all its resources are Available
When the Dispatcher reserves the package
  And the database connection is lost before the transaction commits
  And the server is restarted
Then all resources of the package shall be Available
  And the task shall be in the Ready state
```

#### AT-12 — Access by an unauthorized role
```gherkin
Given a user is logged in as a Field Crew Member
When the user requests the SLA report
Then the system shall reject the request with the message "Access denied"
  And the attempt shall be recorded in the audit log
```

#### AT-13 — Reservation usability
```gherkin
Given a trained Dispatcher is on the status screen of an event with a Ready task
When the Dispatcher reserves the task's package manually
Then the reservation shall be completed in at most 4 UI actions and within 30 seconds
```
