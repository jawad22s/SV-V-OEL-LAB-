# TASK #01: Requirements Extraction

## 1. Functional Requirements

Functional requirements describe what the system shall do.

| Req. ID | Requirement | Type | Priority | Verification Method |
|---|---|---|---|---|
| FR-01 | The system shall check the command format, robot ID, and operator login before accepting a command. After CR-03, it shall also check the operator's role. | FR | High | Command tests: TC-01 to TC-04 |
| FR-02 | The robot shall execute a valid command only once and report the result using the same command ID. | FR | High | Repeat-command test: TC-05 |
| FR-03 | The robot shall deliver the assigned medicine batch to the correct clinic and return to the warehouse after delivery. | FR | High | Delivery test: TC-06 |
| FR-04 | The robot shall stop when it detects an obstacle and continue only when the path is clear. | FR | High | Obstacle test: TC-07 |
| FR-05 | The chamber shall record temperature, humidity, vibration, door status, and power at least once every second. | FR | High | Sensor test: TC-08 |
| FR-06 | The chamber shall activate the required temperature or humidity control when readings go outside the allowed range. | FR | High | Chamber test: TC-08 |
| FR-07 | The robot shall enter Safe Mode when a critical battery, temperature, or fault condition is detected. | FR | **Critical** | Safety tests: TC-09, TC-10 |
| FR-08 | The system shall send robot status to Mission Control, save events during link loss, and send them when the link returns. | FR | High | Link test: TC-11 |
| FR-09 | The robot shall request railway crossing permission and enter only when permission is valid and local safety checks pass. | FR | **Critical** | Crossing test: TC-12 |
| FR-10 | The static portal shall show approved robot details, operator roles, and project documents without exposing passwords. | FR | Medium | Portal check: TC-14 |

---

## 2. Non-Functional Requirements

Non-functional requirements describe the quality, performance, security, reliability, scalability, and usability of the system.

**Note:** The 12-second reply target, recovery times, and usability target are suggested values.

| Req. ID | Requirement | Type | Priority | Verification Method |
|---|---|---|---|---|
| NFR-01 | **Performance:** With a 5-second delay each way, at least 95 out of 100 command replies shall reach Mission Control within 12 seconds. | NFR | High | Timing test: TC-01, TC-13 |
| NFR-02 | **Security:** All commands from users who are not logged in shall be rejected. After CR-03, commands outside the user's role shall also be rejected. | NFR | High | Security tests: TC-02, TC-03 |
| NFR-03 | **Scalability:** After CR-02, the system shall support at least 20 robots connected at the same time without mixing their commands or status. | NFR | High | Load test: TC-13 |
| NFR-04 | **Reliability:** The robot shall retain all event records during a 60-minute link failure and send them within 10 minutes after reconnection at the agreed link speed. | NFR | High | Recovery test: TC-11 |
| NFR-05 | **Usability:** At least 9 out of 10 trained operators shall assign a mission and identify a critical alarm within 2 minutes without help or incorrect commands. | NFR | Medium | User test: TC-15 |

---

## 3. High-Risk Requirements (Safety Requirements)

Safety requirements define the measures needed to prevent accidents, protect medicines, and ensure safe robot operation.

| Req. ID | Safety Requirement | Type | Priority | Verification Method |
|---|---|---|---|---|
| SAF-01 | After CR-01, the robot shall enter Safe Mode within 3 seconds if chamber or battery temperature exceeds the critical limit, or battery capacity falls below the emergency level. | Safety | **Critical** | Timed test: TC-09 |
| SAF-02 | The robot shall start braking within 0.5 seconds of detecting an obstacle and stop within its tested safe stopping distance. | Safety | **Critical** | Braking test: TC-07 |
| SAF-03 | The robot shall reject expired or repeated motion commands and use its local safety response after 15 seconds without a valid heartbeat. | Safety | **Critical** | Command/link tests: TC-04, TC-05, TC-11 |
| SAF-04 | The robot shall not enter a railway crossing without valid permission. If permission is lost before entry, it shall wait outside the tracks. | Safety | **Critical** | Crossing test: TC-12 |

---

## 4. Why These Requirements Are High Risk

The following requirements are considered high risk because failures could cause physical harm, medicine damage, or unsafe robot operation.

- **Safe Mode failure:** A delayed Safe Mode response can damage medicines or cause a power failure.
- **Braking failure:** Slow braking can cause a collision.
- **Communication failure:** Network or heartbeat problems can lead to unsafe commands.
- **Railway crossing failure:** Entering a railway crossing without valid permission can cause a serious accident.

### Important Safety Notes

1. The 0.5-second braking limit and 15-second heartbeat limit are suggested values and must be validated through safety testing.
2. The 3-second Safe Mode response time starts when the critical limit is crossed, not when Mission Control receives the notification.
3. If a problem occurs while the robot is already on the railway tracks, it must follow an approved crossing safety plan. Simply stopping on the tracks may be unsafe.

---

## 5. Summary

This requirements extraction defines the functional capabilities, quality attributes, and safety constraints of the medicine delivery robot system.

- **Functional Requirements:** 10
- **Non-Functional Requirements:** 5
- **Safety Requirements:** 4

**Total Requirements: 19**

The requirements provide a foundation for system design, implementation, testing, and safety verification.

---

*Document: Task #01 — Requirements Extraction*  
*Format: GitHub Markdown (`README.md`)*
