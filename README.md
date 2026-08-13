# Service Call Escalation Spec

A machine readable schema and written specification for deciding what happens to an inbound service call, in the seconds after it arrives.

Maintained by [CIVADOS](https://civados.com/). Published because most escalation logic lives in someone's head, in a dispatcher's memory, or scattered across a phone tree, a CRM automation, and an after hours answering service that do not agree with each other.

---

## The problem this solves

An industrial contractor answers calls that are not equivalent. A blast freezer down at a food plant on a Friday afternoon is not the same event as a quote request for preventive maintenance, and it must not enter the same queue.

Most shops know this. Very few have written it down. The result is predictable:

- The after hours service uses different urgency rules than the daytime office, so the same caller gets a different outcome depending on when they dial.
- Escalation is a phone number, not a ladder. If the first technician does not answer, nothing happens next.
- Nobody can say how long a Priority 1 call waited last month, because priority was never recorded as a field.
- When the automation is finally built, it encodes whatever the person building it assumed that week.

This repository defines the decision as data, so it can be reviewed by the people who own the consequences before it is wired into anything.

---

## What is here

```
schema/escalation-matrix.schema.json   JSON Schema (2020-12) for a matrix document
SPEC.md                                 The written specification and field semantics
examples/                               Worked matrices for real contractor types
```

The schema is the contract. `SPEC.md` explains the reasoning, which matters more than the syntax.

---

## Core model

An inbound contact is classified along four independent axes, and the combination resolves to exactly one routing decision.

| Axis | Question it answers |
|---|---|
| `asset_criticality` | What breaks downstream if this stays broken |
| `service_condition` | Is it down, degraded, or planned |
| `relationship` | Contract customer, warranty, or first time caller |
| `time_context` | Business hours, after hours, weekend, or holiday |

Resolution produces a `priority`, a `response_target`, an ordered `escalation_ladder` with explicit timeouts, and a `fallback` that is required rather than optional.

The fallback requirement is deliberate. An escalation ladder whose last rung is "leave a voicemail" is a ladder that ends in a dropped opportunity, and the spec makes you say so out loud.

---

## Minimal example

```yaml
version: "1.0"
organization: "Example Industrial Refrigeration"
timezone: "America/New_York"

rules:
  - id: p1-production-down-contract
    match:
      asset_criticality: [production_critical]
      service_condition: [down]
      relationship: [contract]
    priority: 1
    response_target:
      acknowledge_within_seconds: 60
      human_contact_within_minutes: 15
    escalation_ladder:
      - notify: on_call_technician
        via: [voice, sms]
        wait_seconds: 300
      - notify: service_manager
        via: [voice]
        wait_seconds: 300
      - notify: owner
        via: [voice]
        wait_seconds: 600
    fallback:
      action: answering_service_live_transfer
      never: [voicemail_only]
```

The illustration above is YAML for readability. Three complete worked matrices in JSON are in [`examples/`](./examples/): [`industrial-refrigeration.json`](./examples/industrial-refrigeration.json) covers production loss, intermittent faults after hours, contract degradation, first time callers, and planned maintenance; [`compressed-air.json`](./examples/compressed-air.json) covers plant air loss, dryer failures that carry moisture into product and tooling without ever showing as down, and pressure drop on a redundant bank versus a single unit site; [`controls-integrator.json`](./examples/controls-integrator.json) covers a line down from a PLC fault, faults during a commissioning window, remote diagnosis attempted before any truck rolls, and ladders that name the engineer assigned to a program rather than any available technician.

---

## How to use this

1. Read [`SPEC.md`](./SPEC.md) and decide whether the four axes describe your operation. If they do not, change them. This is a starting point, not a standard body.
2. Write your matrix as YAML or JSON and validate it against the schema.
3. **Review it with your dispatchers before implementing anything.** They already know where the current rules fail. The document is worth more as a conversation than as a config file.
4. Implement it in whatever platform you use. The schema is deliberately platform neutral and names no vendor.

### Validating

```bash
npx ajv-cli@5 validate --spec=draft2020 -s schema/escalation-matrix.schema.json -d examples/industrial-refrigeration.json
```

Or with Python:

```bash
pip install jsonschema && python -c "import json;from jsonschema import Draft202012Validator as V;V(json.load(open('schema/escalation-matrix.schema.json'))).validate(json.load(open('examples/industrial-refrigeration.json')));print('valid')"
```

If you author in YAML, convert to JSON before validating. The schema itself is format agnostic.

---

## Scope and limits

This spec covers **routing decisions for inbound service contacts**. It does not cover dispatch optimization, technician scheduling, parts availability, or pricing. Those are real problems and they are not this problem.

It assumes a contractor with fewer than roughly fifty field technicians. Above that, the escalation ladder concept usually gives way to a formal dispatch desk and this model will feel thin.

Nothing here is a safety system. If a call path is part of a life safety or regulatory response obligation, that obligation governs and this document does not.

---

## Contributing

Corrections and additional worked examples are welcome, particularly from dispatchers and service managers. See [`CONTRIBUTING.md`](./CONTRIBUTING.md).

If you implement this and something in the model breaks down in practice, that is the most useful thing you can report. Open an issue describing what your operation does that the four axes could not express.

---

## License

MIT. Use it, fork it, ship it commercially, no attribution required.
