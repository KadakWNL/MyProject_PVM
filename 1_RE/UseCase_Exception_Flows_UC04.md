# Use-Case Exception Flow Specification

**Remote Patient Vitals Alert & Monitoring App — Problem Statement #16**

**Use Case:** UC-04 — Escalate Alert to Caregiver Matrix
**Primary Actor:** On-Call Caregiver
**Supporting Actors:** System (Alert Engine), Emergency Dispatch Service
**Related Use Cases:** includes UC-03 (Generate Critical Alert); extended by UC-06 (Notify Emergency Dispatch)
**Related Document:** `UseCase_Flow_UC04.md` (Main Success Scenario and Alternate Flow)

---

## Alternate Flow vs. Exception Flow

This document covers only **exception flows** for UC-04. It is a companion to, and does not replace, the alternate flow already defined in `UseCase_Flow_UC04.md`.

- **Alternate Flow** — a valid, expected system path that the use case can take instead of the main success scenario. For UC-04, this is Tier-1 not acknowledging in time, which leads to Tier-2 escalation and, if that also times out, to UC-06 (Notify Emergency Dispatch). Nothing has gone wrong here; the system is working as designed.
- **Exception Flow** — an abnormal or error condition where the expected system operation cannot proceed normally (e.g., a service outage, a network failure, or a failed write to storage). The system must detect the failure, respond safely, and ensure the alert is never silently lost.

---

## E1 — Caregiver Notification Failure

- **Exception ID:** E1
- **Exception Name:** Caregiver Notification Failure
- **Trigger/Condition:** The system attempts to push a Tier-1 or Tier-2 alert notification (app push + SMS, per FR-002) to the on-call caregiver's device, but the push/SMS gateway returns a delivery failure or the caregiver's registered device is unreachable.
- **System Response:** The Alert Engine marks the notification attempt as "Delivery Failed" instead of "Delivered," and the acknowledgment timer for that tier continues to run from its original start time (the failure does not pause or reset the 3-minute timer).
- **Recovery/Retry Behavior:** The system retries delivery up to 2 additional times over the next 30 seconds using an alternate channel if configured (e.g., SMS if push failed). If all retries fail, the notification is treated as undelivered for that tier.
- **Escalation Behavior:** Because the timer was not paused, an undelivered notification still expires at 3 minutes and triggers the normal escalation path in the alternate flow (Tier-1 → Tier-2 → UC-06), so a silent delivery failure cannot indefinitely stall the alert.
- **Logging/Audit Behavior:** Each delivery attempt (channel, timestamp, result) is written to the alert log, and a "Notification Failure" audit entry is raised so administrators can review caregiver-matrix reachability later.
- **End State:** The alert remains active and continues through the normal escalation timeline; the caregiver matrix entry is flagged for administrator review if repeated failures occur for the same caregiver.

---

## E2 — Network/Communication Failure

- **Exception ID:** E2
- **Exception Name:** Network/Communication Failure
- **Trigger/Condition:** The core system loses connectivity to the caregiver notification service itself (not just one delivery attempt) — e.g., the notification microservice is unreachable or times out.
- **System Response:** The alert is not left unsent. It is written to a local durable queue with status "Pending Dispatch — Comms Down," so the alert exists in persistent storage even though it has not yet reached a caregiver.
- **Recovery/Retry Behavior:** The system polls the notification service on a short interval (e.g., every 15 seconds) and automatically flushes the queued alert as soon as connectivity is restored, preserving the original alert timestamp for timer purposes.
- **Escalation Behavior:** If connectivity is not restored before the queued alert's timer would have expired, the system escalates directly to the next tier (or to UC-06 if no tiers remain) using whatever channel is available, rather than waiting indefinitely for the notification service to recover.
- **Logging/Audit Behavior:** The outage window (start/end), every queued alert affected, and the eventual delivery or escalation outcome are all recorded in the audit log so no alert can be shown to have been "silently dropped."
- **End State:** The alert is either delivered late once connectivity returns, or escalated on schedule — it is never lost, only delayed and clearly logged as delayed.

---

## E3 — Emergency Dispatch Service Unavailable

- **Exception ID:** E3
- **Exception Name:** Emergency Dispatch Service Unavailable
- **Trigger/Condition:** UC-06 (Notify Emergency Dispatch) is triggered per FR-005 after all caregiver tiers time out, but the external Emergency Dispatch Service does not respond or returns an error.
- **System Response:** The system does not treat the failed call as "dispatch notified." The incident status is set to "Dispatch Attempt Failed" instead of "Escalated to Emergency Services," and the patient's location/vitals snapshot is held ready for resend.
- **Recovery/Retry Behavior:** The system retries the dispatch request immediately, then on a short backoff schedule (e.g., at 10s, 30s, 60s) up to a fixed number of attempts, resending the same patient location and current vitals snapshot each time.
- **Escalation Behavior:** If retries continue to fail, the system escalates internally rather than to another external service: it raises a high-priority alert to the System Administrator / clinical operations channel so a human can intervene while automated dispatch attempts continue in the background.
- **Logging/Audit Behavior:** Every dispatch attempt (timestamp, response/error) is logged, and the incident is tagged "Emergency Dispatch Unreachable" for post-incident audit, since this is the last line of defense (FR-005) and must be traceable in detail.
- **End State:** The incident stays open with status "Dispatch Attempt Failed — Retrying" (or "Administrator Notified" if retries are exhausted) until a dispatch confirmation is finally received or a human operator manually resolves the incident.

---

## E4 — Acknowledgement Recording Failure

- **Exception ID:** E4
- **Exception Name:** Acknowledgement Recording Failure
- **Trigger/Condition:** A caregiver taps "Acknowledge" (UC-05) on an active alert, but the system fails to persist the acknowledgment (caregiver ID + timestamp) to the alert log, e.g., due to a database write failure.
- **System Response:** The system does **not** stop the escalation timer on an unconfirmed write. The alert remains in "Pending Acknowledgment" state, and the caregiver's app displays an "Acknowledgment Not Confirmed — Retrying" message rather than a success confirmation, so the caregiver is aware their action has not yet been recorded.
- **Recovery/Retry Behavior:** The system retries the write immediately and, if it succeeds within a short window (e.g., a few seconds), silently reconciles the alert to "Acknowledged" without any escalation having occurred.
- **Escalation Behavior:** If the write cannot be confirmed before the acknowledgment timer would otherwise expire, the alert is allowed to escalate normally to the next tier per the alternate flow (per FR-002/FR-003) — an unconfirmed acknowledgment must never silently block or falsely stop escalation, since that would risk an alert going unanswered.
- **Logging/Audit Behavior:** The failed write attempt, all retries, and the final outcome (confirmed acknowledgment vs. escalation proceeding) are all recorded, so an audit can later show whether a caregiver's acknowledgment action was ever actually lost.
- **End State:** Either the acknowledgment is eventually persisted and the incident closes normally, or the write never confirms in time and the alert escalates — in both cases the caregiver's action and its outcome are fully traceable in the log, with no unrecorded or ambiguous state.

---

## Cross-Reference to Requirements

| Requirement | Relevance to Exception Flows |
|---|---|
| FR-002 | Escalation timing must hold even under E1/E2/E4 — a delivery or write failure cannot pause the 3-minute timer. |
| FR-003 | E4 directly concerns failure to record a caregiver acknowledgment and how the system prevents false/incorrect escalation state. |
| FR-005 | E3 concerns failure of the Emergency Dispatch notification itself, the last line of defense described in FR-005. |
