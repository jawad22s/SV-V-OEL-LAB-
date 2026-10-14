# TASK #02: Change Requests, Testing & Traceability

## Table of Contents

- [1. Change Requests](#1-change-requests)
- [2. Supporting IDs](#2-supporting-ids)
- [3. Constraints](#3-constraints)
- [4. Test Cases — Part A](#4-test-cases--part-a-commands-and-robot-behaviour)
- [5. Test Cases — Part B](#5-test-cases--part-b-safety-and-system-quality)
- [6. Requirement Traceability Matrix](#6-requirement-traceability-matrix-rtm)
- [7. Tests to Re-run](#7-tests-to-re-run-after-changes)
- [8. Conclusion](#8-conclusion)

---

## 1. Change Requests

### Change Log: Version 1.0 to Version 1.1

The following requirements have been modified in Version 1.1 to make them more specific, measurable, and testable.

| Change Request / Requirement | Old Requirement (v1.0) | New Requirement (v1.1) | Status / Reason |
|---|---|---|---|
| CR-01 / FR-07 | The robot shall enter Safe Mode when a critical battery or thermal condition is detected. | The robot shall enter Safe Mode within 3 seconds when chamber or battery temperature exceeds the critical limit, or battery capacity falls below the emergency level. Faults shall still trigger Safe Mode. | Modified — Adds a clear time limit and trigger conditions. |
| CR-02 / NFR-03 | The system shall support communication with multiple robots simultaneously. | The system shall support at least 20 simultaneously connected robots without mixing their commands or status. | Modified — Defines an exact minimum number of robots. |
| CR-03 / FR-01 | Only authenticated operators shall be permitted to issue robot commands. | Before accepting any command, the system shall check that the operator is logged in and that the operator's role allows that command. Command format and robot ID shall still be checked. | Modified — Adds role-based permission checks alongside login verification. |

### 1.1 Why the Changes Are Easier to Test

- **CR-01:** Use a timer to verify that Safe Mode starts within 3 seconds. Test each temperature and battery condition separately.
- **CR-02:** Connect 20 robots simultaneously and verify that all communicate correctly without mixing commands or status.
- **CR-03:** Test a logged-in user with permission and another logged-in user without permission. Only the authorised user's command should be accepted.

---

## 2. Supporting IDs

The following state, transition, and operation IDs are temporary because Tasks 3 to 5 were not provided.

### 2.1 States and Transitions

| State ID | State | Transition |
|---|---|---|
| S01 | Idle | TR01: Valid mission → Navigating |
| S02 | Navigating | TR02: Obstacle detected → Obstacle Avoidance; TR04: Clinic reached → Delivering |
| S03 | Obstacle Avoidance | TR03: Path clear → Previous travel state |
| S04 | Delivering | TR05: Delivery completed → Returning |
| S05 | Returning | TR06: Warehouse reached → Idle |
| S06 | Safe Mode | TR07: Critical condition → Safe Mode; TR11: Safe and authorised recovery → Idle |
| S07 | Waiting at Crossing | TR08: Approach → Wait; TR09: Valid permission and safety checks → Crossing |
| S08 | Crossing | TR10: Tracks cleared → Previous travel state |

### 2.2 Operations

| Operation ID | Action |
|---|---|
| OP01 / OP02 | Check command and permission / Execute once and report result |
| OP03 / OP04 | Deliver and return / Stop and avoid obstacle |
| OP05 / OP06 | Read sensors / Correct chamber conditions |
| OP07 / OP08 | Enter or recover from Safe Mode / Save and send events |
| OP09 / OP10 | Check crossing permission / Publish portal |
| OP11 / OP12 | Manage robot connections / Assign mission and identify alarm |

---

## 3. Constraints

### Rules Used in the Requirement Traceability Matrix

A constraint is a rule that the system must follow during operation.

| Constraint ID | Rule |
|---|---|
| C01 | Accept commands only after checking login, role permission, command format, and robot ID. Report and log rejected commands. |
| C02 | Execute each command ID no more than once. Reject expired commands and commands that are not allowed in the current state. |
| C03 | Deliver to the assigned clinic, allow handover only to a verified recipient, and return to Idle at the warehouse. |
| C04 | Start braking within the suggested 0.5 seconds. Continue only when the path is clear. |
| C05 | Read all five sensors every second and correct temperature or humidity when necessary. |
| C06 | Enter Safe Mode within 3 seconds of a CR-01 trigger. Recovery requires permission and cleared safety checks. |
| C07 | Enter railway tracks only with valid permission and local safety checks. If a problem occurs on the tracks, follow the approved crossing plan. |
| C08 | After 15 seconds without a heartbeat, use the local safety response. Preserve events during a 60-minute outage and send all events within 10 minutes of reconnection at the agreed link speed. |
| C09 | Support at least 20 connected robots and keep their data separate. With a 5-second delay each way, at least 95% of replies should arrive within 12 seconds. |
| C10 | Publish approved portal information only. Do not publish passwords or live command controls. |
| C11 | At least 9 out of 10 trained users must complete the specified mission and alarm tasks within 2 minutes without help or incorrect commands. |

---

## 4. Test Cases — Part A: Commands and Robot Behaviour

Each command should follow this workflow:

`Received → Authenticated and Authorised → Executed → Status Reported → Logged`

Invalid commands must be rejected before execution. Their rejection must also be reported and logged.

| Test ID | Test | Steps | Expected Result |
|---|---|---|---|
| TC-01 | Valid Command | A logged-in operator with permission sends a valid mission command. | Receive the command, check login and role, execute once, report status, and log the result. The reply meets the suggested timing target. |
| TC-02 | Not Logged In | Send a command without a valid login. | Reject without robot movement. Report and log the rejection reason. |
| TC-03 | Wrong Role | A logged-in user without motion permission sends a motion command. | Reject without movement. Report and log the rejection reason. |
| TC-04 | Invalid Command | Separately send expired commands, incorrect robot IDs, and incorrectly formatted commands. | Reject each invalid command and log the reason. |
| TC-05 | Repeated Command | Send the same command ID twice. Also send a command that is invalid in the current state. | Execute the first valid command only once. Reject the repeated execution and state-invalid command, and log the results. |
| TC-06 | Delivery | Start a valid mission, reach the clinic, verify the recipient, and complete delivery. | Deliver the correct batch, return to the warehouse, enter Idle, and report and log the steps. |
| TC-07 | Obstacle | Place an obstacle in the path during outbound travel and return travel. | Start braking within 0.5 seconds, stop safely, and continue only after the path is clear. Report and log the event. |
| TC-08 | Chamber | Check all five sensors and set temperature or humidity outside the allowed ranges. | Record readings every second, activate the appropriate correction, and report and log the event. |

---

## 5. Test Cases — Part B: Safety and System Quality

| Test ID | Test | Steps | Expected Result |
|---|---|---|---|
| TC-09 | Safe Mode Timing | Separately raise chamber and battery temperatures above critical limits and lower battery capacity below its emergency level. Repeat without a communication link. | Enter Safe Mode within 3 seconds for every applicable trigger. Verify that equality with the limit and safe values do not trigger it by themselves. Save local timestamps. |
| TC-10 | Fault and Recovery | Create a controller fault. Attempt recovery with incorrect permission and while safety checks are failing. Finally, clear the checks and use an authorised operator. | Enter Safe Mode on fault. Reject unsafe recovery and allow authorised recovery only after safety checks pass. Log every result. |
| TC-11 | Link Failure | Apply a 5-second delay, remove the heartbeat for more than 15 seconds, and disconnect for 60 minutes. Restore the agreed link speed. | Apply the local safety response, preserve all events, and send them within 10 minutes after reconnection. Verify that Mission Control receives every event. |
| TC-12 | Railway Crossing | Test absent permission, expired permission, valid permission, link loss before entry, and a fault while crossing. | Wait outside for invalid permission. Enter only with valid permission and successful local checks. Follow the approved plan for an in-crossing fault and log results. |
| TC-13 | 20 Robots | Connect 20 robots and send separate commands to each. Attempt a 21st connection. | All 20 robots remain connected with correct data and timed replies. Accept or reject the 21st robot safely without disturbing existing connections. |
| TC-14 | Portal | Open the robot, operator, and project document pages. | Approved information is available, links work, and passwords are not displayed. |
| TC-15 | Ease of Use | Ask 10 trained operators to assign a mission and identify a critical alarm. | At least 9 operators finish within 2 minutes without help or incorrect commands. |

> **Testing note:** These are planned tests, not completed test results. Suggested limits must be approved before testing the real system.

---

## 6. Requirement Traceability Matrix (RTM)

The Requirement Traceability Matrix links each requirement to its state or transition, operation, constraint, test case, and risk level.

| Requirement | State / Transition | Operation | Constraint | Test Case | Risk |
|---|---|---|---|---|---|
| FR-01 | All states | OP01 | C01, C02 | TC-01–TC-04 | High |
| FR-02 | TR01 / S02 | OP02 | C02 | TC-01, TC-05 | High |
| FR-03 | TR01, TR04–TR06 | OP03 | C03 | TC-06 | High |
| FR-04 | TR02–TR03 / S03 | OP04 | C04 | TC-07 | Critical |
| FR-05 | All active states | OP05 | C05 | TC-08 | High |
| FR-06 | Chamber control | OP06 | C05 | TC-08 | High |
| FR-07 | TR07, TR11 / S06 | OP07 | C06 | TC-09, TC-10 | Critical |
| FR-08 | Link lost / restored | OP08 | C08 | TC-11 | High |
| FR-09 | TR08–TR10 / S07–S08 | OP09 | C07 | TC-12 | Critical |
| FR-10 | Portal publication | OP10 | C10 | TC-14 | Medium |
| NFR-01 | Command / reply | OP02, OP08 | C09 | TC-01, TC-13 | High |
| NFR-02 | Login / role check | OP01 | C01 | TC-02, TC-03 | High |
| NFR-03 | 20 connections | OP11 | C09 | TC-13 | High |
| NFR-04 | Link lost / restored | OP08 | C08 | TC-11 | High |
| NFR-05 | Mission / alarm | OP12 | C11 | TC-15 | Medium |
| SAF-01 | TR07 / S06 | OP07 | C06 | TC-09 | Critical |
| SAF-02 | TR02 / S03 | OP04 | C04 | TC-07 | Critical |
| SAF-03 | Command / link loss | OP01, OP08 | C02, C08 | TC-04, TC-05, TC-11 | Critical |
| SAF-04 | TR08–TR10 / S07–S08 | OP09 | C07 | TC-12 | Critical |

---

## 7. Tests to Re-run After Changes

Regression testing verifies that a change has not broken previously working functionality.

| Change Request | Tests to Re-run | Reason |
|---|---|---|
| CR-01 | TC-09, TC-10, TC-11, TC-12 | Verify the 3-second Safe Mode limit, fault response, local safety during link loss, and railway crossing safety. |
| CR-02 | TC-13, TC-01, TC-11 | Verify 20 simultaneous connections, command replies under load, and event recovery after link restoration. |
| CR-03 | TC-01, TC-02, TC-03, TC-05, TC-10, TC-15 | Verify allowed commands, login failures, role restrictions, repeated commands, recovery permissions, and operator usability. |

---

## 8. Conclusion

In this assignment, I reviewed 10 functional requirements, 5 non-functional requirements, and 4 high-risk safety requirements. I also documented three change requests, defined supporting IDs and constraints, prepared 15 planned test cases, and developed a Requirement Traceability Matrix.

The changes improve testability by defining a 3-second Safe Mode response, requiring at least 20 simultaneous robot connections, and enforcing both authentication and role-based authorisation for commands. The regression test plan identifies which tests must be repeated when these requirements change.

Since Tasks 3 to 5 were not provided, the supporting state, transition, and operation IDs are temporary. Actual temperature limits, battery thresholds, and the railway crossing safety plan must be confirmed before implementation and physical safety testing.

---

**Document:** Task #02 — Change Requests, Testing & Traceability  
**Version:** 1.1  
**Format:** GitHub Markdown  
**Testing Status:** Planned — execution results not yet recorded
