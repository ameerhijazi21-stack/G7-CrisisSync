## AT-03 — Assigned Field Commander reports completion (normal flow)

Use case: UC-06 Report Task Completion

```gherkin
Given Commander A is signed in and assigned to lead Task A
And Task A is Active and has reserved resources
And Task B is Blocked because Task A is its only unfinished prerequisite
When Commander A reports completion of Task A
Then Task A becomes Completed
And its reserved resources are released and become available
And Task B becomes Ready
```

## AT-04 — Another Field Commander tries to complete the task (error flow)

Use case: UC-06 Report Task Completion

```gherkin
Given Commander B is signed in
And Task A is Active, has reserved resources, and is assigned to Commander A
And Task B is Blocked because Task A is its only unfinished prerequisite
When Commander B attempts to report completion of Task A
Then CrisisSync rejects the attempt and displays an authorization message
And Task A remains Active
And its resources remain reserved
And Task B remains Blocked
When Commander A signs in and reports completion of Task A
Then Task A becomes Completed
And its reserved resources are released and become available
And Task B becomes Ready
```
