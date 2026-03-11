# Digital Product Engineering – Test 9 Solution

## Principal Delivery Manager — Release R2: Digital Front Door Platform

---

## SECTION 1 – INTEGRATED DELIVERY SCHEDULE + CRITICAL PATH

### Scheduling Methodology

- **Start Date:** Monday, April 6, 2026
- **Business days only** (Mon–Fri)
- **Stream Capacity Constraints:**
  - Stream A: 1 task at a time
  - Stream B: 2 tasks at a time
  - Stream C: 1 task at a time

### Stream Scheduling Decisions

**Stream B (2 concurrent slots):**

- W1 runs alone (Days 1–3), then W3 + W4 fill both slots (Days 4–9/4–8). When W4 completes (Day 8), W10 takes the freed slot (Days 9–12). W11 follows after all three predecessors complete (Day 13). W12 follows W11. W16 follows W15.

**Stream C (1 at a time):**

- Order: W2 → W8 → W13 → W5. This prioritizes W13 (which feeds the critical W14) over W5 (a leaf node).

**Stream A (1 at a time):**

- Order: W6 → W7 → W14 → W15 → W9. Non-critical leaf tasks (W6, W7) are front-loaded into the idle period before W14 becomes dependency-ready. W9 (also a leaf) is deferred until after the critical W15.

### Integrated Delivery Schedule

| ID  | Work Item                      | Stream | Duration | Start Date | End Date | Bus. Days |
| --- | ------------------------------ | ------ | -------- | ---------- | -------- | --------- |
| W1  | Finalize API contracts         | B      | 3        | Apr 6      | Apr 8    | 1–3       |
| W2  | Consent policy rules           | C      | 4        | Apr 6      | Apr 9    | 1–4       |
| W3  | Build booking API adapter      | B      | 6        | Apr 9      | Apr 16   | 4–9       |
| W4  | Build secure messaging adapter | B      | 5        | Apr 9      | Apr 15   | 4–8       |
| W5  | Notification routing rules     | C      | 4        | Apr 22     | Apr 27   | 13–16     |
| W6  | Mobile booking UI              | A      | 7        | Apr 9      | Apr 17   | 4–10      |
| W7  | Web booking UI                 | A      | 6        | Apr 20     | Apr 27   | 11–16     |
| W8  | Auth integration (OIDC)        | C      | 5        | Apr 10     | Apr 16   | 5–9       |
| W9  | Consent capture UI             | A      | 5        | May 8      | May 14   | 25–29     |
| W10 | Event bus + DLQ setup          | B      | 4        | Apr 16     | Apr 21   | 9–12      |
| W11 | Integration test harness       | B      | 5        | Apr 22     | Apr 28   | 13–17     |
| W12 | Performance testing            | B      | 3        | Apr 29     | May 1    | 18–20     |
| W13 | Security review                | C      | 3        | Apr 17     | Apr 21   | 10–12     |
| W14 | UAT and defect triage          | A      | 5        | Apr 29     | May 5    | 18–22     |
| W15 | Release readiness              | A      | 2        | May 6      | May 7    | 23–24     |
| W16 | Production cutover             | B      | 1        | May 8      | May 8    | 25        |

### Overall Release Completion Date: **May 8, 2026**

_(All leaf work items including W9 complete by May 14, 2026)_

### Critical Path (Resource-Constrained)

The critical path is the longest chain of dependent tasks whose delay directly delays the project end date:

**W1 → W4 (frees B slot) → W10 → W11 → W14 → W15 → W16**

| Step | Task     | Rationale                                                                    |
| ---- | -------- | ---------------------------------------------------------------------------- |
| 1    | W1 (3d)  | All downstream B tasks depend on W1                                          |
| 2    | W4 (5d)  | Must complete to free the second B slot for W10                              |
| 3    | W10 (4d) | Last predecessor of W11 to finish (Day 12); gates W11 start                  |
| 4    | W11 (5d) | Gates both W12 (performance) and W14 (UAT)                                   |
| 5    | W14 (5d) | Finishes Day 22; W12 finishes Day 20; W14 is the binding predecessor for W15 |
| 6    | W15 (2d) | Final readiness gate before cutover                                          |
| 7    | W16 (1d) | Production cutover                                                           |

**Total critical path duration: 25 business days**

### Tasks with < 2 Days Slack

| Task | Scheduled Finish | Latest Finish | Slack (days) |
| ---- | ---------------- | ------------- | ------------ |
| W1   | Day 3 (Apr 8)    | Day 3         | **0**        |
| W4   | Day 8 (Apr 15)   | Day 8         | **0**        |
| W10  | Day 12 (Apr 21)  | Day 12        | **0**        |
| W11  | Day 17 (Apr 28)  | Day 17        | **0**        |
| W14  | Day 22 (May 5)   | Day 22        | **0**        |
| W15  | Day 24 (May 7)   | Day 24        | **0**        |
| W16  | Day 25 (May 8)   | Day 25        | **0**        |

All seven critical-path tasks have **zero slack**. Any 1-day slip on any of these tasks delays the release.

**Near-critical (2 days slack):** W12 — Performance testing finishes Day 20 but is not needed until Day 23 (W15 start); however, W15 is gated by W14 (Day 22), giving W12 only 2 days of float.

---

## SECTION 2 – INTEGRATION DEPENDENCY ORCHESTRATION

### Dependency Matrix

| Interface | Description                      | Work Items Responsible            | Ready Date | Test Type                           |
| --------- | -------------------------------- | --------------------------------- | ---------- | ----------------------------------- |
| I1        | Front Door → API Gateway         | W3, W6, W7                        | Apr 27     | Contract test, E2E API test         |
| I2        | Gateway → EHR Scheduling         | W3                                | Apr 16     | Integration test (mock + live)      |
| I3        | Gateway → Secure Messaging       | W4                                | Apr 15     | Integration test, message flow      |
| I4        | Gateway → Event Bus              | W3, W10                           | Apr 21     | Async integration, DLQ validation   |
| I5        | Event Bus → Notification Service | W5, W10                           | Apr 27     | Async E2E, delivery confirmation    |
| I6        | Front Door → Identity/Consent    | W8, W9, W2                        | May 14     | OIDC flow test, consent enforcement |
| I7        | Notification → SMS Provider      | W5                                | Apr 27     | Rate limit test, delivery test      |
| I8        | Front Door → Televisit Platform  | _ASSUMPTION: Separate workstream_ | N/A        | Integration test                    |

### Integration Order: I6 → I1 → I2 → I4 → I5

| Order | Interface                                 | Rationale                                                                                                                                                                                                                |
| ----- | ----------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1st   | **I6** — Front Door → Identity/Consent    | Authentication and consent must be established first. No user action (booking, messaging) can proceed without a verified identity and consent grant. OIDC tokens are prerequisites for all downstream API calls.         |
| 2nd   | **I1** — Front Door → API Gateway         | With identity established, the front door must be able to route authenticated requests to the API gateway. The gateway acts as the single entry point for all backend services; nothing downstream functions without it. |
| 3rd   | **I2** — Gateway → EHR Scheduling         | The core use case is appointment booking. Once the gateway can receive requests (I1), the booking flow through the EHR scheduling system must be validated. This produces the booking events that feed downstream.       |
| 4th   | **I4** — Gateway → Event Bus              | Booking confirmations, cancellations, and other domain events must be published to the event bus. I4 depends on I2 because events are triggered by booking transactions processed through the gateway.                   |
| 5th   | **I5** — Event Bus → Notification Service | Patient notifications (SMS, email) are driven by events on the bus. I5 cannot be tested until I4 is proven to publish events correctly. This is the final link in the end-to-end booking-to-notification chain.          |

---

## SECTION 3 – RAID LOG WITH QUANTIFIED RISK SCORING

### Risk Scoring Formula

**Risk Score = Probability × Impact × Detectability**

Scale: 1 (low) – 5 (high) for each dimension

### Risks (sorted by highest score)

| ID  | Risk Description                                                                                      | Prob | Impact | Detect | Score  | Mitigation                                                                                     | Owner            |
| --- | ----------------------------------------------------------------------------------------------------- | ---- | ------ | ------ | ------ | ---------------------------------------------------------------------------------------------- | ---------------- |
| R1  | EHR scheduling API timeouts under load cause booking failures exceeding 2% threshold (F1)             | 4    | 5      | 3      | **60** | Implement circuit breaker with cached slot fallback; pre-test with 2× expected RPM             | Stream B Lead    |
| R2  | Event bus processing failure causes DLQ overflow, losing notifications and booking confirmations (F5) | 3    | 5      | 4      | **60** | DLQ replay automation with ≤30-min recovery SLA; DLQ size alerting at 1,000 messages           | Stream B Lead    |
| R3  | Consent service mismatch blocks patient booking/messaging due to policy rule errors (F4)              | 3    | 4      | 3      | **36** | Pre-validation logic in consent capture; automated consent rule regression suite               | Stream C Lead    |
| R4  | Notification provider throttling delays patient alerts beyond acceptable window (F3)                  | 4    | 4      | 2      | **32** | Queue with rate limiter; secondary SMS provider failover route                                 | Stream C Lead    |
| R5  | Identity Platform Upgrade dependency (May 15) delays production release past May 12 target            | 3    | 5      | 2      | **30** | Backward compatibility layer in OIDC integration; escalate to Identity team for early delivery | Delivery Manager |

### Top 3 Risks and Mitigation Owners

| Rank | Risk                          | Score | Mitigation Owner | Action                                                                                                 |
| ---- | ----------------------------- | ----- | ---------------- | ------------------------------------------------------------------------------------------------------ |
| 1    | R1 — EHR scheduling timeouts  | 60    | Stream B Lead    | Implement circuit breaker pattern in W3; validate during W12 performance testing with 120+ booking RPM |
| 2    | R2 — Event bus / DLQ overflow | 60    | Stream B Lead    | Design DLQ replay in W10; validate recovery time ≤30 min during W11 integration testing                |
| 3    | R3 — Consent service mismatch | 36    | Stream C Lead    | Add pre-validation rules in W2; test consent enforcement in W14 UAT                                    |

### Assumptions

| ID  | Assumption                                                                                                                      | Impact if Wrong                                                     |
| --- | ------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------- |
| A1  | **ASSUMPTION:** EHR scheduling system will be available for integration testing by Apr 22 (W11 start)                           | W11 blocked; critical path delayed; release date at risk            |
| A2  | **ASSUMPTION:** Vendor televisit platform integration (I8) is out of scope for R2 and will be delivered in a subsequent release | If in scope, additional work items and integration testing required |

### Issues

| ID  | Issue                                                                                                                                  | Impact                                                                                                                     | Action Required                                                            |
| --- | -------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| IS1 | Stream A capacity constraint (1 task at a time) forces sequential execution of W6 and W7, consuming 13 business days for UI work alone | Stream A idle for 1 day (Day 17) waiting for W14 dependencies; leaf tasks W6, W7, W9 delay but do not affect critical path | Monitor; escalate if A team can temporarily increase to 2 concurrent tasks |
| IS2 | W12 (Performance testing) has only 2 days of slack; any delay to W11 compresses or eliminates the performance testing buffer           | Unvalidated performance risks violating the <1.2s P95 latency production goal                                              | Track W11 daily; prepare parallel performance test environment in advance  |

### Dependencies

| ID  | Dependency                                                       | Type     | Required By                | Status                                               |
| --- | ---------------------------------------------------------------- | -------- | -------------------------- | ---------------------------------------------------- |
| D1  | Identity Platform Upgrade must complete for production OIDC auth | External | May 12 (release readiness) | At risk — external team targets May 15 (3 days late) |

---

## SECTION 4 – RELEASE READINESS AND GO/NO-GO CRITERIA

### Release Readiness Checklist

| #   | Category                 | Criteria                                                                                      | Measurement                                             | Validated By                   |
| --- | ------------------------ | --------------------------------------------------------------------------------------------- | ------------------------------------------------------- | ------------------------------ |
| 1   | API Contract Validation  | All 8 integration interfaces pass contract tests                                              | 100% contract test pass rate in CI                      | W11 (Integration test harness) |
| 2   | Rate Limiting Validation | SMS provider throttling tested at 2× peak load (300 notification RPM)                         | Zero dropped notifications under throttle conditions    | W12 (Performance testing)      |
| 3   | Idempotency Validation   | Duplicate event handling verified with idempotency keys across booking and notification flows | 0 duplicate bookings; ≤0.5% duplicate notifications     | W11 + W12                      |
| 4   | Consent Enforcement      | Consent rules enforced for all patient interactions; unauthorized actions blocked             | 100% consent checks pass; 0 unauthorized actions in UAT | W14 (UAT)                      |
| 5   | DLQ Replay Procedure     | DLQ replay recovers missed events within SLA; runbook validated                               | Recovery time ≤30 minutes from 5,000-message backlog    | W11 (Integration test)         |
| 6   | Performance Validation   | P95 latency and booking success rate under production-equivalent load                         | P95 <1.2s; booking success ≥98% at 120 RPM              | W12 (Performance testing)      |
| 7   | Security Clearance       | Zero critical/high vulnerabilities from security review                                       | 0 open critical/high findings                           | W13 (Security review)          |
| 8   | UAT Sign-off             | All P1/P2 defects resolved; business scenarios pass                                           | 100% P1/P2 closure; all UAT scenarios passed            | W14 (UAT)                      |

### Go/No-Go Decision Gates

| Gate                        | Metric                          | Threshold     | Data Source                    | Action if Not Met                                                            |
| --------------------------- | ------------------------------- | ------------- | ------------------------------ | ---------------------------------------------------------------------------- |
| G1: API Contracts           | Contract test pass rate         | 100%          | CI pipeline (W11)              | **No-Go** — Fix failing contracts; retest                                    |
| G2: Booking Success         | Booking success rate under load | ≥ 98%         | Performance test results (W12) | **No-Go** — Profile EHR integration; optimize circuit breaker; retest        |
| G3: P95 Latency             | Booking P95 latency             | ≤ 1.2 seconds | Performance test results (W12) | **No-Go** — Identify hot paths; optimize caching/query; retest               |
| G4: Auth Failures           | Authentication failure rate     | ≤ 0.3%        | OIDC integration test (W11)    | **No-Go** — Debug OIDC flow; check token refresh; retest                     |
| G5: Duplicate Notifications | Duplicate notification rate     | ≤ 0.5%        | E2E notification test (W11)    | **No-Go** — Validate idempotency keys; check event dedup; retest             |
| G6: DLQ Recovery            | DLQ replay recovery time        | ≤ 30 minutes  | DLQ replay drill (W11)         | **No-Go** — Optimize replay consumer; increase throughput; retest            |
| G7: Security                | Critical/High vulnerabilities   | 0 open        | Security review report (W13)   | **No-Go** — Remediate all critical/high findings before cutover              |
| G8: UAT Sign-off            | P1/P2 defects resolved          | 100% resolved | UAT defect tracker (W14)       | **No-Go** — Triage remaining defects; fix or defer with stakeholder approval |

### Go/No-Go Decision Process

- **Go:** All 8 gates pass thresholds → Proceed to W16 (Production cutover)
- **Conditional Go:** G5 or G6 marginally missed (within 10% of threshold) → Proceed with enhanced monitoring and rollback plan
- **No-Go:** Any gate fails significantly → Delay cutover; assign fix owners; retest within 2 business days

---

## SECTION 5 – EXECUTIVE STATUS SUMMARY

### R2 Digital Front Door — Executive Status Update

**Report Date:** April 6, 2026 (Project Kickoff)
**Target Production Cutover:** May 8, 2026
**Overall Status: 🟢 GREEN — On Track**

---

#### Status by Workstream

| Stream                                | Status   | Summary                                                                                                                                  |
| ------------------------------------- | -------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| **Stream A** (UI / UAT)               | 🟢 Green | W6 (Mobile booking UI) begins Apr 9. UAT (W14) planned Apr 29–May 5. All tasks sequenced within capacity.                                |
| **Stream B** (Platform / Integration) | 🟢 Green | W1 (API contracts) starts Apr 6. Critical path runs through B (W1→W4→W10→W11→W12→W16). Two-task concurrency fully utilized in Weeks 1–2. |
| **Stream C** (Policy / Security)      | 🟢 Green | W2 (Consent rules) and W8 (OIDC auth) begin Apr 6 and Apr 10. Security review (W13) completes Apr 21.                                    |

#### Top 3 Risks

| #   | Risk                                                                                       | Score | Mitigation                                                                           |
| --- | ------------------------------------------------------------------------------------------ | ----- | ------------------------------------------------------------------------------------ |
| 1   | EHR scheduling timeouts → booking failure rate >2%                                         | 60    | Circuit breaker + cached fallback; validate during performance testing (W12)         |
| 2   | Event bus failure → DLQ overflow → lost notifications                                      | 60    | DLQ replay automation; ≤30-min recovery SLA; validated in W11                        |
| 3   | Identity Platform Upgrade completes May 15 — 3 days after our May 12 full-readiness target | 30    | Backward compatibility layer in W8; escalation to Identity team for earlier delivery |

#### Key Delivery Metrics

| Metric                                     | Target             | Current Status                      |
| ------------------------------------------ | ------------------ | ----------------------------------- |
| Schedule adherence                         | 100% tasks on time | On track (Day 0)                    |
| Critical path slack                        | 0 days             | Confirmed — 7 tasks with zero slack |
| Production goal: Booking failure rate      | < 2%               | To be validated in W12 (Apr 29)     |
| Production goal: P1 incidents first 72 hrs | 0                  | To be validated post-cutover        |
| Production goal: P95 latency               | < 1.2 seconds      | To be validated in W12 (Apr 29)     |

#### Next Milestones

| Milestone                               | Date      | Dependency               |
| --------------------------------------- | --------- | ------------------------ |
| API contracts finalized (W1)            | Apr 8     | None — first deliverable |
| Booking API adapter complete (W3)       | Apr 16    | W1                       |
| Integration test harness complete (W11) | Apr 28    | W3, W4, W10              |
| Performance testing complete (W12)      | May 1     | W11                      |
| UAT complete (W14)                      | May 5     | W11, W13                 |
| Release readiness sign-off (W15)        | May 7     | W12, W14                 |
| **Production cutover (W16)**            | **May 8** | W15                      |

#### Potential Surprises

1. **Identity Platform Upgrade (External):** Scheduled May 15 — 3 days after our full readiness target. If backward compatibility is not feasible, cutover may shift to May 16.
2. **Notification Vendor Migration (External):** Completing May 10 — 2-day buffer. Any vendor migration slip risks I5 and I7 notification interfaces.
3. **Stream A bottleneck:** Single-task capacity means W9 (Consent capture UI) cannot start until May 8. While W9 is not on the critical path, it is required for full I6 (Identity/Consent) integration validation.

---

## SECTION 6 – MULTI-REGION ROLLOUT STRATEGY (PRINCIPAL SECTION)

### Regional Traffic Profile (Dataset E)

| Region    | Traffic % | Booking RPM | Notification RPM |
| --------- | --------- | ----------- | ---------------- |
| Rochester | 50%       | 60          | 150              |
| Arizona   | 30%       | 36          | 90               |
| Florida   | 20%       | 24          | 60               |

### Phased Rollout Plan

| Stage                      | Region                          | Traffic % | Duration | Success Criteria                                                                                                                         |
| -------------------------- | ------------------------------- | --------- | -------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| **1. Internal Validation** | Rochester (internal staff only) | ~1%       | 2 days   | 0 P1 incidents; P95 latency <1.2s; all 8 integration interfaces functional; booking success ≥99% on internal traffic                     |
| **2. Pilot Deployment**    | Florida                         | 20%       | 3 days   | Booking success ≥98%; duplicate notifications ≤0.5%; auth failures ≤0.3%; DLQ size stable (<100 messages); 0 P1 incidents                |
| **3. Controlled Ramp**     | Florida + Arizona               | 50%       | 3 days   | All metrics sustained at combined 60 booking RPM / 150 notification RPM; P95 <1.2s; DLQ replay tested under live traffic; 0 P1 incidents |
| **4. Full Rollout**        | All regions (add Rochester)     | 100%      | 2 days   | All metrics sustained at full load (120 booking RPM / 300 notification RPM); 72-hour stability window begins; 0 P1 incidents             |

### Rollout Rationale

- **Florida first** (20% traffic) — smallest traffic footprint minimizes blast radius. Validates all integration points under real patient traffic with lowest risk.
- **Add Arizona** (30%) — doubles traffic volume; validates scaling behavior and notification throughput at 150 RPM.
- **Rochester last** (50%) — highest traffic region added only after two stages of production validation. Full load confirms system meets all performance targets.

### Rollback Criteria

If at any stage:

- Booking success rate drops below 95%, OR
- P1 incident is declared, OR
- DLQ size exceeds 5,000 messages

→ **Immediately halt rollout**, roll back to previous stage, and initiate incident response (Section 9).

---

## SECTION 7 – FAILURE INJECTION / RESILIENCE TESTING

### Failure Injection Test Plan

| #   | Test Scenario                    | Trigger                                                                                          | Expected Behavior                                                                                                                                                                                          | Metric                                                                                                            |
| --- | -------------------------------- | ------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| 1   | **Event Replay**                 | Kill event processor mid-batch; strand 500 events in DLQ                                         | DLQ replay mechanism detects orphaned events and republishes them; consumer processes replayed events; no duplicate notifications due to idempotency keys                                                  | Recovery time ≤30 min; 0 duplicate notifications; 100% event recovery                                             |
| 2   | **API Throttling**               | Inject HTTP 429 (Too Many Requests) responses from EHR scheduling API at 50% rate for 10 minutes | Circuit breaker activates after threshold failures; retry queue absorbs burst; booking requests served from cache or queued for retry; no permanent failures                                               | Booking success ≥98%; P95 latency <1.2s; circuit breaker trips within 30s                                         |
| 3   | **Notification Provider Outage** | Block all outbound REST calls to SMS provider for 15 minutes                                     | Notifications queued in retry buffer; no messages lost; queue drains after provider recovery; patients receive delayed but complete notifications                                                          | 0 lost notifications; delivery delay ≤20 min; queue fully drained within 10 min of recovery                       |
| 4   | **Consent Service Delay**        | Inject 5-second latency on all consent/identity service responses for 10 minutes                 | Pre-validation cache serves cached consent decisions for known patients; new/uncached patients experience degraded but functional flow; auth failure rate stays within SLA                                 | Auth failures ≤0.3%; P95 consent check latency <3s; 0 unauthorized booking completions                            |
| 5   | **DLQ Overflow**                 | Flood event bus with 10,000 malformed messages in 5 minutes                                      | DLQ accepts overflow without dropping messages; alerting triggers at 1,000-message threshold; healthy/well-formed events continue processing on separate partition; malformed events isolated for analysis | DLQ alert fires within 2 min; healthy event processing unaffected; 0 data loss; DLQ drain initiated within 15 min |

### Execution Schedule

- Tests 1–3: Execute during W11 (Integration test harness) — Apr 22–28
- Tests 4–5: Execute during W12 (Performance testing) — Apr 29–May 1
- All tests must pass before G6 (DLQ Recovery) and G2 (Booking Success) release gates

---

## SECTION 8 – PORTFOLIO DEPENDENCY DECISION ANALYSIS

### External Program Dependencies

| Program                       | Completion Date | R2 Release Readiness | Gap                 |
| ----------------------------- | --------------- | -------------------- | ------------------- |
| Identity Platform Upgrade     | May 15          | May 12               | **+3 days late** ⚠️ |
| Notification Vendor Migration | May 10          | May 12               | −2 days (OK) ✓      |

### 1. Impact Analysis

**Identity Platform Upgrade (May 15) — CRITICAL CONFLICT**

- R2 uses OIDC authentication (W8, Interface I6) for all patient-facing interactions.
- If the Identity Platform Upgrade introduces breaking changes to OIDC endpoints, token formats, or consent APIs, R2 cannot safely go live on May 12.
- A 3-day gap means either R2 launches on the old identity platform (risking incompatibility) or delays to May 16+.
- **Impact:** Potential 4-day release delay (to May 16), affecting all three regions' go-live schedule.

**Notification Vendor Migration (May 10) — MEDIUM RISK**

- R2's notification interfaces (I5, I7) depend on the SMS provider integration.
- Migration completes 2 days before release readiness, leaving minimal buffer for regression validation.
- If migration slips by even 2 days, notification integration is unvalidated at go-live.
- **Impact:** Notification delivery reliability uncertain; possible violation of ≤0.5% duplicate notification target.

### 2. Options to Resolve

| Option                              | Description                                                                                      | Pros                                         | Cons                                                                     |
| ----------------------------------- | ------------------------------------------------------------------------------------------------ | -------------------------------------------- | ------------------------------------------------------------------------ |
| **A. Delay R2 to May 16**           | Wait for Identity Platform Upgrade completion (May 15); go live May 16                           | Zero dependency risk; clean integration      | 4-day schedule impact; stakeholder expectations reset                    |
| **B. Accelerate Identity Upgrade**  | Negotiate with Identity team to complete upgrade by May 9                                        | No R2 delay; clean integration               | Depends on external team; may not be feasible                            |
| **C. Backward compatibility layer** | Implement abstraction in W8 (OIDC integration) supporting both old and new identity platform     | Removes hard dependency; R2 launches on time | ~2 days additional development in W8; added testing complexity           |
| **D. Partial release**              | Launch non-identity-dependent features May 12; add identity-dependent features after May 15      | Partial value delivered on time              | Feature fragmentation; complex rollout; patient experience inconsistency |
| **E. Dual notification path**       | For Notification Vendor Migration: maintain old SMS provider as fallback during migration window | Eliminates notification risk                 | Additional configuration; old provider costs continue briefly            |

### 3. Recommended Approach

**Primary: Option C (Backward compatibility) + Option E (Dual notification path)**

1. **Add a backward compatibility abstraction layer in W8** that supports both the current and upgraded Identity Platform OIDC endpoints. This decouples R2's release date from the Identity team's schedule. Estimated effort: +2 days to W8, which has sufficient float (W8 has ~5 days of float in the current schedule).
2. **Maintain the existing SMS provider as a fallback** during the Notification Vendor Migration window. Configure feature flag to switch between old and new providers. Minimal effort; adds resilience.
3. **Fallback: Option A** — If during W8 implementation the backward compatibility layer proves architecturally infeasible (e.g., breaking schema changes), immediately escalate and plan for May 16 cutover. Communicate the potential delay to stakeholders by Apr 20 at the latest.

**Decision Timeline:**

- By Apr 16 (W8 complete): Confirm backward compatibility is feasible → proceed with May 8 cutover
- If infeasible by Apr 16: Trigger Option A → communicate May 16 revised cutover to stakeholders

---

## SECTION 9 – INCIDENT COMMAND RESPONSE PLAN

### Incident Scenario

During launch, the system experiences:

- **Booking success rate drops to 94%** (target: ≥98%)
- **Notification delay reaches 22 minutes** (normal: <1 min)
- **DLQ size grows to 8,000 messages** (normal: ~0)

### 30-Minute Incident Response Plan

| Minute | Action                                                                                                                                                                                                                                     | Responsible Team                                |
| ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------- |
| 0–2    | **Declare P1 incident.** Open war room (bridge call). Page on-call leads for Streams A, B, C. Assign Incident Commander, Communications Lead, and Scribe.                                                                                  | Incident Commander (Delivery Manager)           |
| 2–5    | **Triage dashboards.** Confirm booking failure rate (94%), notification delay (22 min), DLQ size (8,000). Identify which regions are affected. Check if metrics are worsening or stable.                                                   | Stream B Lead (Platform)                        |
| 5–8    | **Investigate EHR integration.** Check EHR scheduling API response times, error rates (HTTP 5xx/429), and circuit breaker state. Query booking service logs for timeout patterns.                                                          | Stream B (Integration Team)                     |
| 8–10   | **Investigate event bus.** Check consumer lag on event bus topics, DLQ growth rate (messages/min), and dead-letter reasons. Check notification provider API status and response times.                                                     | Stream B (Event Bus) + Stream C (Notifications) |
| 10–13  | **Apply EHR mitigation.** If EHR timeouts confirmed: activate circuit breaker failover mode; enable cached scheduling slot fallback for high-demand appointment types; reduce booking retry concurrency to prevent thundering herd.        | Stream B (Integration Team)                     |
| 13–16  | **Apply DLQ mitigation.** Initiate DLQ replay for recoverable messages (valid schema, retryable errors). Pause non-critical event consumers to prioritize booking confirmations and notification events.                                   | Stream B (Event Bus Team)                       |
| 16–19  | **Apply notification mitigation.** If notification provider degraded: activate backup SMS route (secondary provider). Throttle non-urgent/marketing notifications. Prioritize booking confirmation and appointment reminder notifications. | Stream C (Notifications Team)                   |
| 19–22  | **Validate recovery.** Monitor booking success rate — is it trending toward 98%? Check DLQ drain rate — is the backlog decreasing? Verify notification delivery latency dropping below 5 minutes.                                          | Stream B Lead + Stream C Lead                   |
| 22–25  | **Assess rollback.** If booking success rate not above 96% by minute 22, prepare rollback plan: redirect traffic to previous release; halt multi-region rollout progression. If recovering, continue monitoring.                           | Incident Commander                              |
| 25–28  | **Stakeholder communication.** Send executive incident notification with: current status, root cause hypothesis, mitigation actions taken, expected recovery time. Update status page for patient-facing impact.                           | Communications Lead                             |
| 28–30  | **Decision point.** **Continue** if all three metrics are trending toward thresholds. **Rollback** if any metric is still degrading. Log incident timeline, actions taken, and assign post-incident review within 48 hours.                | Incident Commander                              |

### Top 5 Root Cause Hypotheses (Ranked by Likelihood)

| Rank | Hypothesis                                                                                                                                                                             | Supporting Evidence                                                                                                                        | Investigation                                                                                     |
| ---- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------- |
| 1    | **EHR scheduling API degradation** causing booking timeouts → cascading event backlog → DLQ growth → notification delays                                                               | Booking failure rate (94%) is the primary symptom; DLQ and notification issues are downstream effects of failed booking events             | Check EHR API latency p99, error rates, circuit breaker trip count                                |
| 2    | **Event bus consumer failure** stalling message processing → DLQ overflow → unprocessed booking confirmations → undelivered notifications                                              | 8,000-message DLQ suggests consumer-side failure, not publisher-side; booking failures may be secondary (timeout waiting for confirmation) | Check consumer group lag, consumer error logs, last successful offset                             |
| 3    | **Notification provider rate limiting** triggered by traffic spike → backed-up notification queue → event bus backpressure → booking service timeout from full queues                  | 22-minute notification delay indicates downstream bottleneck; if backpressure propagates upstream, booking latency increases               | Check SMS provider response codes (429), notification queue depth, event bus backpressure metrics |
| 4    | **Database connection pool exhaustion** on booking service → failed writes → retry storms → event bus flooding → DLQ overflow                                                          | All three symptoms (booking failures, event backlog, notification delay) can originate from a database bottleneck at the booking layer     | Check DB connection pool utilization, query latency, active connection count                      |
| 5    | **Consent service latency spike** causing auth/consent pre-validation timeouts → booking requests rejected at validation step → failed booking events published → cascading downstream | Booking failures could stem from upstream consent check failures rather than EHR issues                                                    | Check consent service response times, OIDC token validation latency, auth failure rate            |

---

## ASSUMPTIONS LOG (Consolidated)

All assumptions made throughout this document are collected here for transparency:

| ID  | Section   | Assumption                                                                                                                       |
| --- | --------- | -------------------------------------------------------------------------------------------------------------------------------- |
| A1  | Section 2 | **ASSUMPTION:** Vendor televisit platform integration (I8) is out of scope for R2                                                |
| A2  | Section 3 | **ASSUMPTION:** EHR scheduling system available for integration testing by Apr 22                                                |
| A3  | Section 6 | **ASSUMPTION:** Internal validation at Rochester uses staff-only traffic (~1% of total volume)                                   |
| A4  | Section 8 | **ASSUMPTION:** May 12 release readiness date accounts for multi-region rollout preparation after W16 production cutover (May 8) |
| A5  | Section 9 | **ASSUMPTION:** On-call teams for all three streams are staffed and available during launch window                               |

---

_Document prepared for Release R2 — Digital Front Door Platform_
_Delivery Manager: Hameem Mahdi Ph.D., M.S.E_
_Date: March 11, 2026_
