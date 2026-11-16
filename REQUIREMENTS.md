# CrisisSync - Requirements and Acceptance Tests

## Group Information
- Group Number:
- Team Member 1:
  - Name: Joe Abou Ghannam
  - Student ID: 322470998
  - Academic Email: joe.abou.ghannam@e.braude.ac.il
- Team Member 2:
  - Name: Eden Krainits
  - Student ID: 211589742
  - Academic Email: eden.krainits@e.braude.ac.il
- Team Member 3:
  - Name: Dvir Klein
  - Student ID: 318931169
  - Academic Email: dvir.klein@e.braude.ac.il
- Team Member 4:
  - Name: Moshe Ben Shabat
  - Student ID: 206394793
  - Academic Email: moshe.ben.shabat@e.braude.ac.il

---

## 1. Functional Requirements (FR)

### 1.1 Resource Reservation and Task Completion

| ID | Requirement |
|----|-------------|
| FR-01 | The system shall allow a Dispatcher to reserve a resource package for a task only when the task is in the Ready state. |
| FR-02 | The system shall reserve all required resources of the package in a single transaction; if any required resource is unavailable, the system shall reject the entire reservation, keep the task in the Ready state, display a failure message to the Dispatcher, and refresh the Dispatcher's screen to the current resource state. |
| FR-03 | The system shall allow only the assigned Field Commander to report task completion, and shall then transition the task to the Completed state, record the completion time, and immediately release all resources allocated to it. |
| FR-04 | When a task transitions to the Completed state, the system shall transition to the Ready state every dependent task whose prerequisites are all completed. |

### 1.2 SLA Report Generation

| ID | Requirement |
|----|-------------|
| FR-05 | The system shall allow the System Administrator to generate an SLA report for a selected past month, and shall reject a request for the current month or a future month with an error message. |
| FR-06 | The system shall calculate, for the events whose last task reached a final state in the selected month, the average time from event opening to the completion of its last task, grouped by event type, and shall store and display the report. |
| FR-07 | If the event data cannot be retrieved from the database, the system shall store no SLA report, keep existing reports unchanged, and display an error message. |

---

## 2. Non-Functional Requirements (NFR)

| ID | Category | Requirement | Measurement Method |
|----|----------|-------------|--------------------|
| NFR-01 | Performance | The system shall push task and resource status updates to all connected clients within 2 seconds of a status change (95th percentile) while 50 clients are connected. | Load test of 1,000 status changes; latency = client receipt time minus server commit time. |
| NFR-02 | Security | The system shall reject 100% of attempts by Field Commanders to view or complete tasks not assigned to them. | Automated test covering every combination of Field Commander and task assignment. |
| NFR-03 | Reliability | Under 100 simultaneous reservation requests for the same resource, repeated 1,000 times, the system shall produce zero double reservations and zero partial reservations. | Stress test followed by a database query that counts resources reserved for more than one task and partially reserved packages. |
| NFR-04 | Usability | At least 90% of trained Dispatchers shall complete a resource package reservation for a Ready task on their first attempt without assistance. | Usability test with 20 trained Dispatchers. |

---

## 3. Requirements Traceability Matrix (RTM)

| Requirement ID | Use Case | Acceptance Test ID |
|----------------|----------|--------------------|
| FR-01 | Reserve Resource Package | TC-01, TC-02 |
| FR-02 | Reserve Resource Package, Handle Reservation Failure | TC-01, TC-03 |
| FR-03 | Report Task Completion | TC-04, TC-05 |
| FR-04 | Report Task Completion | TC-04 |
| FR-05 | Generate SLA Report | TC-06, TC-07 |
| FR-06 | Generate SLA Report | TC-08, TC-09 |
| FR-07 | Generate SLA Report | TC-10 |
| NFR-01 | Report Task Completion, Track Real-Time Event Status | TC-11 |
| NFR-02 | Report Task Completion, View Assigned Task Details | TC-12 |
| NFR-03 | Reserve Resource Package | TC-13 |
| NFR-04 | Reserve Resource Package | TC-14 |

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


#### TC-03: Atomic Reservation Failure and Recovery

**Given** a task is in the Ready state
And one required resource is unavailable
And all other required resources are available

**When** the Dispatcher requests to reserve the resource package

**Then** the system shall reject the entire reservation
And no resources from the requested package shall be reserved
And the task shall remain in the Ready state
And the Dispatcher shall see a failure message
And the Dispatcher's screen shall be refreshed to the current resource state


#### TC-04: Successful Task Completion

**Given** a task is in the Active state
And exactly one Field Commander is assigned to the task
And the task has allocated resources
And a dependent task is in the Blocked state
And all other prerequisites of the dependent task are completed

**When** the assigned Field Commander reports task completion

**Then** the system shall transition the task to the Completed state
And the completion time shall be recorded
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

**Then** the system shall generate and store the SLA report for the selected month
And the report shall be displayed to the System Administrator


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


#### TC-10: Data Retrieval Failure and Recovery

**Given** the System Administrator has selected a past month
And the database connection fails during data retrieval

**When** the System Administrator requests to generate the SLA report

**Then** the system shall store no SLA report
And previously stored reports shall remain unchanged
And an error message shall be displayed


### 4.3 Non-Functional Acceptance Tests

#### TC-11: Real-Time Status Synchronization

**Given** 50 clients are connected to the system

**When** 1,000 task and resource status changes are committed

**Then** at least 95% of the changes shall be received by every client within 2 seconds of the commit


#### TC-12: Unauthorized Task Access Prevention

**Given** every combination of Field Commander and task assignment is generated
And each Field Commander requests to view and to complete every task

**When** the system processes these requests

**Then** the system shall reject 100% of the requests for tasks not assigned to the requesting Field Commander
And no unauthorized task details shall be disclosed
And no task status or allocated resources shall be changed


#### TC-13: Concurrent Resource Reservation Consistency

**Given** 100 tasks are in the Ready state
And all of them require the same non-shareable resource
And only one unit of that resource is available

**When** 100 reservation requests for that resource are sent simultaneously
And the test is repeated 1,000 times

**Then** in every repetition exactly one reservation shall succeed
And all other reservations shall be rejected
And the consistency query shall find zero resources reserved for more than one task
And the consistency query shall find zero partially reserved packages


#### TC-14: Resource Reservation Usability

**Given** 20 trained Dispatchers participate in a usability test
And each Dispatcher is assigned a Ready task with all required resources available

**When** each Dispatcher attempts to reserve a resource package for the first time without assistance

**Then** at least 18 out of 20 Dispatchers shall successfully complete the reservation
And no assistance shall be required for those successful attempts
