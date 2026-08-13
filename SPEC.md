# Service Call Escalation Specification

Version 1.0

This document explains the reasoning behind the schema. The schema enforces structure. This explains why the structure is shaped the way it is, which is the part that determines whether an implementation survives contact with a real dispatch desk.

---

## 1. Design principles

**1.1 The decision is data, not code.** Escalation logic embedded in a phone tree or an automation builder cannot be reviewed by the people who understand the consequences. Expressed as a document, it can be read by a service manager in ten minutes and corrected before anything is wired up.

**1.2 Roles, never people.** Every `notify` value is a role. Matrices that name individuals decay the moment someone changes shift, takes leave, or leaves the company, and the decay is silent.

**1.3 Every path must terminate in a commitment.** There is no rule without a `fallback`, and plain voicemail is not an available fallback action. If a ladder can be exhausted, the caller must still receive something that confirms a human will follow up.

**1.4 Unknown is a classification, not an absence.** `unknown` is a valid value on the criticality and relationship axes. Forcing a classifier to guess produces confident wrong answers, and a call it admits it could not classify is safer than one it mislabeled.

**1.5 Four priorities, no more.** Operations that define six or seven levels stop honoring the distinction under load, and the lower levels collapse into a single ignored bucket. Four is a constraint chosen for behavior, not for expressiveness.

---

## 2. The four axes

The axes are independent by design. Any dependency between them belongs in a rule, not in the vocabulary.

### 2.1 `asset_criticality`

Judged by **downstream consequence**, not by equipment size, cost, or age.

| Value | Meaning |
|---|---|
| `production_critical` | Failure stops production, spoils product, or halts shipping |
| `production_impacting` | Failure reduces throughput or forces a workaround, but output continues |
| `ancillary` | Comfort, convenience, or non production systems |
| `unknown` | The caller could not say and the classifier could not infer |

A small compressor feeding a single critical process line is `production_critical`. A large rooftop unit conditioning an office is `ancillary`. Contractors consistently get this backwards when they classify by equipment rather than by consequence.

### 2.2 `service_condition`

| Value | Meaning |
|---|---|
| `down` | Not functioning |
| `degraded` | Functioning outside specification |
| `intermittent` | Functioning now, failing unpredictably |
| `planned` | Scheduled work, no current fault |
| `inquiry` | Question, quote, or new business |

**`intermittent` is deliberately separate from `degraded`.** This is the most consequential modeling decision in the spec. Intermittent faults are routinely routed as low urgency because the asset is working when the technician calls back. On a critical asset, an intermittent fault is frequently the last warning before a full failure, and the operation discovers it the following morning.

### 2.3 `relationship`

| Value | Meaning |
|---|---|
| `contract` | Under a service agreement with committed response terms |
| `warranty` | Covered by warranty obligation |
| `existing` | Prior customer, no active agreement |
| `new` | First contact |
| `unknown` | Not resolvable at classification time |

**A contract does not outrank a physical emergency.** Contract terms set a *floor* on response, not a filter on who gets answered. A `new` caller with a `production_critical` asset `down` must not be deprioritized behind a `contract` customer with a `degraded` ancillary unit. The example matrix enforces this by matching its priority 1 rule on criticality and condition only, ignoring relationship entirely.

### 2.4 `time_context`

Derived from `timezone` and `business_hours`. Never from server time. Assuming server time is the most common implementation defect in this entire model, and it surfaces as calls being classified as after hours during the working day.

---

## 3. Resolution

1. Classify the contact on all four axes.
2. Walk `rules` **in document order**.
3. A rule matches when, for every axis present in its `match` block, the classified value is in that axis list. Axes absent from `match` are not considered.
4. The first matching rule wins. Evaluation stops.
5. If no rule matches, `default_rule` applies.

Order is significant. Specific rules go first. This is a deliberate choice over a scoring system: scoring is harder to audit, and a service manager must be able to trace why a given call went where it went.

### 3.1 Why `default_rule` should not be priority 4

An unclassified call is an **unknown risk**, not a low one. Defaulting it to the lowest priority means the calls the system understood least receive the least attention. The example sets its default to priority 2 so a human resolves the ambiguity quickly.

---

## 4. Response targets

`acknowledge_within_seconds` is the number that governs whether the caller keeps dialing your competitors. It may be satisfied by an automated response, provided the response is specific. A generic hold message is not an acknowledgement.

`human_contact_within_minutes` requires a person who can make a commitment. An automated system confirming receipt does not satisfy it.

`onsite_target_hours` is optional and should be omitted unless it reflects a real commitment. An aspirational figure here becomes a breached target in reporting, and the usual response is to stop running the report.

---

## 5. Escalation ladders

Each rung notifies a role through one or more channels, then waits `wait_seconds` before advancing.

- `wait_seconds: 0` notifies and advances immediately, producing a parallel broadcast.
- Email alone is not an escalation channel at priority 1 or 2.
- Ladders should end at an authority who can commit resources. A ladder whose last rung is another technician who may also be on a roof has not escalated anything.

**Ladder timing should reflect observed response, not intent.** If your on call technician realistically answers in seven minutes, a 120 second rung guarantees the ladder advances past a person who was about to pick up.

---

## 6. Anti-patterns this spec makes visible

| Anti-pattern | How the spec surfaces it |
|---|---|
| Emergencies ending in voicemail | Plain voicemail is not a valid `fallback.action` |
| Escalation that is a single phone number | `escalation_ladder` requires at least one ordered rung and `fallback` is mandatory |
| Named individuals in routing | `notify` is documented as a role |
| Priority invented per call | Priority is a resolved output of the matrix |
| After hours rules disagreeing with daytime rules | Both live in one document under one `timezone` |
| Unclassified calls silently deprioritized | `default_rule` is required and explicit |

---

## 7. Implementation notes

**Log the rule `id` on every contact.** Without it, no reporting question can be answered. With it, you can measure response against target per rule and find which rule is wrong. Rule ids are therefore stable identifiers and must not be renamed once in production.

**Classification confidence belongs in your implementation, not in the matrix.** If an automated classifier is unsure, route to `unknown` and let `default_rule` handle it.

**Test the ladder by exhausting it.** Most implementations are never tested past the first rung. Run a drill that lets a priority 1 fall all the way through to `fallback`, out of hours, and watch what actually happens.

---

## 8. Versioning

`version` refers to this specification, not to the document. Breaking changes to field names or enum values increment the major version. Additive changes increment the minor version. Implementations should reject a `version` whose major component they do not support.

---

## 9. Non goals

This spec does not cover dispatch optimization, technician scheduling, routing by geography, parts availability, or pricing.

It is not a safety system. Where a call path forms part of a life safety or regulatory response obligation, that obligation governs and nothing in this document modifies it.
