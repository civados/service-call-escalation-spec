# Security Policy

See the [CIVADOS organization security policy](https://github.com/civados/.github/blob/main/SECURITY.md) for reporting instructions, response targets, and scope.

**Do not open a public issue for a security report.** Use GitHub private vulnerability reporting on this repository, or email security@civados.com.

## Specific to this repository

This repository publishes routing logic as data. The most valuable report you can make here is not a conventional software vulnerability.

**In scope, and actively wanted:**

- A matrix that validates cleanly against `schema/escalation-matrix.schema.json` but can still silently drop an emergency contact.
- An ambiguity in `SPEC.md` that would lead two competent implementers to build materially different routing from the same document.
- A gap in the CI checks that lets an unsafe example merge.
- Any defect in the schema that permits a rule with no terminating commitment.

The design intent is that **no valid matrix can produce a dropped emergency call**. If you can construct a counterexample, that is a defect in the specification and we want it reported.

**Out of scope:**

- Style, wording, and modeling preferences. Open a normal issue for those.
- Vulnerabilities in the automation platforms that consume matrices built from this spec. Report those to the platform vendor.
