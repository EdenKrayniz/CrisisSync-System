# CrisisSync - Requirements and Acceptance Tests

## Group Information
- Group Number:
- Team Member 1:
    - Name: joe abou ghannam
    - Student ID:322470998
    - Academic Email: joe.abou.ghannam@e.braude.ac.il
- Team Member 2:
    - Name: eden krainits
    - Student ID: 211589742
    - Academic Email: eden.krainits@e.braude.ac.il
- Team Member 3:
  - Name: dvir klein
  - Student ID: 318931169
  - Academic Email: dvir.klein@e.braude.ac.il
- Team Member 4:
  - Name: mosh ben shabat
  - Student ID: 206394793
  - Academic Email: moshe.ben.shabat@e.braude.ac.il



---

## 1. Functional Requirements (FR)

### 1.1 Resource Reservation and Task Completion

| ID | Requirement |
|----|-------------|
| FR-01 |The system shall allow a Dispatcher to reserve a resource package for a task only when the task is in the Ready state. |
| FR-02 |The system shall reserve all required resources atomically and reject the entire reservation if any required resource is unavailable. |
| FR-03 |The system shall allow the assigned Field Commander to report task completion, transition the task to the Completed state, immediately release all resources allocated to it, and update dependent tasks to the Ready state when all their prerequisites are completed. |

### 1.2 SLA Report Generation

| ID | Requirement |
|----|-------------|
| FR-04 |The system shall allow the System Administrator to generate an SLA report for a selected past month. |
| FR-05 |The system shall calculate the average time from event opening to the completion of its last task, grouped by event type, in the SLA report. |

---


## 2. Non-Functional Requirements (NFR)

| ID | Requirement |
|----|-------------|
| NFR-01 | The system shall synchronize task and resource status updates to all connected clients within 2 seconds of a status change using server push. |
| NFR-02 |The system shall reject 100% of unauthorized attempts by Field Commanders to access or complete tasks not assigned to them. |
| NFR-03 |The system shall maintain 100% resource reservation consistency during concurrent reservation requests, ensuring that no resource is allocated to more than one active task simultaneously. |
| NFR-04 |The system shall allow at least 90% of trained Dispatchers to complete a resource package reservation for a Ready task without assistance on their first attempt. |


---

## 3. Requirements Traceability Matrix (RTM)

| Requirement ID | Use Case | Acceptance Test ID |
|----------------|----------|--------------------|
| FR-01 |Reserve Resource Package |TC-01, TC-02 |
| FR-02 |Reserve Resource Package |TC-01, TC-03 |
| FR-03 |Report Task Completion |TC-04, TC-05 |
| FR-04 |Generate SLA Report |TC-06, TC-07 |
| FR-05 |Generate SLA Report |TC-08, TC-09 |
| NFR-01 |Report Task Completion, Track Real-Time Event Status |TC-10 |
| NFR-02 | Report Task Completion, View Assigned Task Details  |TC-11 |
| NFR-03 |Reserve Resource Package |TC-12 |
| NFR-04 | Reserve Resource Package |TC-13 |

---

## 4. BDD Acceptance Tests

### 4.1 Resource Reservation and Task Completion

#### TC-01: Successful Reservation
**Given** a task is in the Ready state
And all required resources are available

**When** the Dispatcher requests to reserve the resource package

**Then** the system shall reserve all required resources
And the task shall transition to the Active state


#### TC-02: Reservation Rejected for Blocked Task

**Given** a task is in the Blocked state
And all required resources are available

**When** the Dispatcher requests to reserve the resource package

**Then** the system shall reject the reservation request
And the task shall remain in the Blocked state
And all required resources shall remain available

#### TC-03: Atomic Reservation Failure

**Given** a task is in the Ready state
And one required resource is unavailable
And all other required resources are available

**When** the Dispatcher requests to reserve the resource package

**Then** the system shall reject the entire reservation
And no resources from the requested package shall be reserved
And the task shall remain in the Ready state


#### TC-04: Successful Task Completion

**Given** a task is in the Active state
And exactly one Field Commander is assigned to the task
And the task has allocated resources
And a dependent task is in the Blocked state
And all other prerequisites of the dependent task are completed

**When** the assigned Field Commander reports task completion

**Then** the system shall transition the task to the Completed state
And all allocated resources shall be released and become available
And the dependent task shall transition to the Ready state


#### TC-05: Unauthorized Task Completion

**Given** a task is in the Active state
And a Field Commander is assigned to the task
And another Field Commander is not assigned to the task

**When** the unassigned Field Commander attempts to report task completion

**Then** the system shall reject the completion request
And the task shall remain in the Active state
And all allocated resources shall remain reserved



### 4.2 SLA Report Generation


#### TC-06: Successful SLA Report Generation

**Given** the System Administrator is logged into the system
And a past month is selected

**When** the System Administrator requests to generate an SLA report for the selected month

**Then** the system shall generate the SLA report for the selected month
And the report shall be available for the System Administrator to view


#### TC-07: SLA Report Generation for an Invalid Month

**Given** the System Administrator is logged into the system
And the current month is selected

**When** the System Administrator requests to generate an SLA report for the selected month

**Then** the system shall reject the report generation request
And display an error message indicating that only past months can be selected
And no SLA report shall be generated


#### TC-08: Successful SLA Average Time Calculation

**Given** two completed events of the same type are included in the selected month
And the first event took 4 hours from opening to completion of its last task
And the second event took 6 hours from opening to completion of its last task

**When** the System Administrator generates the SLA report

**Then** the system shall calculate an average handling time of 5 hours for that event type
And display the result in the SLA report


#### TC-09: SLA Average Time Calculation with No Completed Events

**Given** a past month is selected
And no completed events of a particular event type exist for that month

**When** the System Administrator generates the SLA report

**Then** the system shall not calculate an average handling time for that event type
And shall not display a misleading numerical average for that event type


### 4.3 Non-Functional Acceptance Tests


#### TC-10: Real-Time Status Synchronization

**Given** multiple clients are connected to the system
And a task is in the Active state
And the task has allocated resources

**When** the assigned Field Commander completes the task

**Then** the system shall update the task status to Completed
And release all allocated resources
And synchronize the updated task status and resource availability to all connected clients using server push within 2 seconds


#### TC-11: Unauthorized Task Access Prevention

**Given** 100 unauthorized access attempts are made by Field Commanders to tasks not assigned to them
And some attempts request to view task details
And other attempts request to report task completion

**When** the system processes these requests

**Then** the system shall reject all 100 unauthorized attempts
And no unauthorized task details shall be disclosed
And no task status or allocated resources shall be changed


#### TC-12: Concurrent Resource Reservation Consistency

**Given** two tasks are in the Ready state
And both tasks require the same non-shareable resource
And only one unit of that resource is available
And two Dispatchers attempt to reserve the resource simultaneously

**When** the system processes both reservation requests

**Then** exactly one reservation shall succeed
And the other reservation shall be rejected
And the shared resource shall be allocated to only one task
And no partial reservation shall remain for the rejected task


#### TC-13: Resource Reservation Usability

**Given** 20 trained Dispatchers participate in a usability test
And each Dispatcher is assigned a Ready task with all required resources available

**When** each Dispatcher attempts to reserve a resource package for the first time without assistance

**Then** at least 18 out of 20 Dispatchers shall successfully complete the reservation
And no assistance shall be required for those successful attempts