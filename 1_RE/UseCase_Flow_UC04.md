# Use-Case Flow Specification

**Remote Patient Vitals Alert & Monitoring App — Problem Statement #16**

**Use Case:** UC-04 — Escalate Alert to Caregiver Matrix
**Primary Actor:** On-Call Caregiver
**Supporting Actors:** System (Alert Engine), Emergency Dispatch Service
**Related Use Cases:** includes UC-03 (Generate Critical Alert); extended by UC-06 (Notify Emergency Dispatch)

## Preconditions
- The patient's wearable/bedside device is actively transmitting vitals and is registered in the monitoring system.
- Clinical thresholds for the patient (SpO2, Heart Rate, BP) have been configured (FR-004).
- At least one On-Call Caregiver is defined in the caregiver escalation matrix for the patient.

## Postconditions
- **Success:** The alert is acknowledged by a caregiver, with caregiver ID and timestamp recorded, and escalation stops.
- **Failure:** All caregiver tiers time out, the alert is forwarded to the Emergency Dispatch Service, and the incident is logged for audit.

## Main Success Scenario
1. System detects a threshold breach during continuous vitals evaluation (UC-02) and generates a critical alert (UC-03), including patient ID, vital reading, and timestamp.
2. System identifies the Tier-1 On-Call Caregiver from the patient's caregiver matrix and pushes a real-time alert notification (app push + SMS).
3. Caregiver receives the notification and opens the alert on the mobile/web app.
4. System displays the patient's live vitals, breach details, and recent trend to the caregiver.
5. Caregiver reviews the alert and taps "Acknowledge" (UC-05).
6. System records the caregiver ID and acknowledgment timestamp, halts further escalation, and marks the alert status as "In Progress."
7. Caregiver takes clinical action and later marks the alert "Resolved"; system logs resolution time and closes the incident.
8. Use case ends successfully.

## Alternate Flow — 2a. Tier-1 Caregiver Does Not Acknowledge In Time
- **2a1.** System starts a 3-minute acknowledgment timer when the Tier-1 notification is sent.
- **2a2.** If no acknowledgment is received within 3 minutes, the system automatically escalates the alert to the Tier-2 caregiver and repeats steps 3–6.
- **2a3.** If Tier-2 also fails to acknowledge within its timeout, the system exhausts the caregiver matrix and triggers UC-06 (Notify Emergency Dispatch), sending the patient's location and current vitals to the Emergency Dispatch Service.
- **2a4.** The incident is flagged as "Escalated to Emergency Services" and remains open until a dispatch confirmation or caregiver acknowledgment is received.
